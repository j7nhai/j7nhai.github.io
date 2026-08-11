---
tags: [AI, spark, lance, arrow]
lang: zh
ref: spark-df-write-to-lance
permalink: /zh/2026/08/11/spark-df-wirte-to-lance.html
---

## ✨ Spark DataFrame 如何通过 `lance-spark` 写入 Lance

### 👋 背景

Spark 从 2.3 开始引入 **DataSource V2**，把外部数据源读写抽象成标准接口（如 `Table`、`ScanBuilder`、`WriteBuilder`）。相比 V1，V2 写入协议更清晰，两阶段提交语义也更标准。

`lance-spark` 的写入就是基于这套协议。用户执行：

```scala
df.write.format("lance").save(path)
```

Spark 会驱动完整的 V2 写入生命周期，最终将数据落为 Lance 列式格式。🚀

---

### 🔧 Spark 提供的底层能力

- 声明能力：`SupportsWrite` / `SupportsTruncate`
- 构建写入：`newWriteBuilder(...)` → `BatchWrite`
- 并行执行：Driver 下发 `DataWriterFactory`，Executor 调 `createWriter(...)`
- 两阶段提交：Executor 返回 `WriterCommitMessage`，Driver `commit(messages)`；失败 `abort(...)`

---

### 🧭 写入总体流程（DataSource V2）

1. `df.write.format("lance").save(path)`
2. 构建 `WriteBuilder` / `BatchWrite`
3. Driver 创建并分发 `DataWriterFactory`
4. Executors 执行 `createWriter(...)` 并行写入
5. Executors 返回 `WriterCommitMessage`
6. Driver 统一 `commit(messages)`（失败时 `abort(...)`）

<div align="center">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 600 440" width="600" height="440">
<rect width="600" height="440" rx="8" fill="#1e1e2e"/>
<rect x="120" y="20" width="120" height="40" rx="6" fill="#313244" stroke="#89b4fa" stroke-width="1.5"/>
<text x="180" y="46" text-anchor="middle" fill="#cdd6f4" font-size="15" font-weight="bold" font-family="monospace">Driver</text>
<rect x="400" y="20" width="120" height="40" rx="6" fill="#313244" stroke="#a6e3a1" stroke-width="1.5"/>
<text x="460" y="46" text-anchor="middle" fill="#cdd6f4" font-size="15" font-weight="bold" font-family="monospace">Executor</text>
<line x1="180" y1="60" x2="180" y2="420" stroke="#585b70" stroke-width="1.5"/>
<line x1="460" y1="60" x2="460" y2="420" stroke="#585b70" stroke-width="1.5"/>
<rect x="40" y="80" width="380" height="95" rx="5" fill="#313244" stroke="#89b4fa" stroke-width="1"/>
<text x="60" y="108" fill="#a6adc8" font-size="14" font-family="monospace">- save(path)</text>
<text x="60" y="133" fill="#a6adc8" font-size="14" font-family="monospace">- 创建 WriteBuilder / BatchWrite</text>
<text x="60" y="158" fill="#a6adc8" font-size="14" font-family="monospace">- createBatchWriterFactory</text>
<line x1="180" y1="200" x2="448" y2="200" stroke="#cdd6f4" stroke-width="1.5"/>
<polygon points="448,196 458,200 448,204" fill="#cdd6f4"/>
<text x="240" y="192" fill="#cdd6f4" font-size="13" font-family="monospace">下发 DataWriterFactory</text>
<rect x="360" y="225" width="200" height="70" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="1"/>
<text x="380" y="253" fill="#a6adc8" font-size="14" font-family="monospace">- createWriter</text>
<text x="380" y="278" fill="#a6adc8" font-size="14" font-family="monospace">- 并行写数据</text>
<line x1="455" y1="325" x2="202" y2="325" stroke="#cdd6f4" stroke-width="1.5" stroke-dasharray="6,4"/>
<polygon points="202,320 192,325 202,330" fill="#cdd6f4"/>
<text x="220" y="317" fill="#cdd6f4" font-size="13" font-family="monospace">返回 WriterCommitMessage</text>
<rect x="120" y="350" width="120" height="45" rx="5" fill="#313244" stroke="#f9e2af" stroke-width="1"/>
<text x="140" y="378" fill="#a6adc8" font-size="14" font-family="monospace">- 提交</text>
</svg>
</div>

---

### 🧩 Spark Schema vs Lance（Arrow）Schema

Lance 底层采用 Arrow Schema，Spark 侧是 `StructType`。两者在基础类型（`Int`/`Long`/`Float`/`String` 等）上可以直接映射，但 Arrow 的类型表达更细。

- **FixedSizeList vs ArrayType**：Arrow 区分定长与变长列表，Spark 只有 `ArrayType`
- **LargeUtf8 / LargeBinary**：Arrow 支持 64 位偏移的大对象类型
- **Float16**：Arrow 支持半精度，Spark 无原生等价类型

在 `lance-spark` 中，这些额外语义通过字段 metadata（如 `arrow.fixed-size-list.size`）保留，再由 `LanceArrowUtils.toArrowSchema()` 还原到 Arrow 类型。✅

---

### 📝 Lance Writer 细节

`LanceArrowWriter` 内部持有一组 field writer，每个 field writer 负责一个列：

```scala
class LanceArrowWriter(root: VectorSchemaRoot, fields: Array[LanceArrowFieldWriter])
```

Executor 侧，Spark 框架逐行传入 `InternalRow`，而 Lance 底层是 Arrow 列式存储。因此每接收一行，`LanceArrowWriter` 会遍历所有 field writer，各自向对应的列向量追加一个元素，完成**行转列**。

---

### ⚙️ Field Writer 细节：以 `FixedSizeListWriter` 为例

#### 🗂️ Arrow FixedSizeList 内存布局

参考 [Arrow Columnar Format](https://arrow.apache.org/docs/format/Columnar.html#fixed-size-list-layout)，一个 `FixedSizeList<Float32>[2048]` 的 vector 由两部分组成：

```
FixedSizeListVector (listSize=2048)
├── validity buffer: 每行 1 bit，标记是否 null
└── child Float4Vector (value buffer): 连续存放所有行的元素
    → row0 的 2048 个 float | row1 的 2048 个 float | ...
```

没有 offset buffer——因为每行的元素个数固定，第 i 行的数据起始位置 = `i * listSize`。

#### 📖 FixedSizeListWriter 源码逻辑

```scala
class FixedSizeListWriter(
    val valueVector: FixedSizeListVector,
    val elementWriter: LanceArrowFieldWriter)  // 这里是 FloatWriter

  def setValue(input: SpecializedGetters, ordinal: Int): Unit = {
    val array = input.getArray(ordinal)            // 从 Spark Row 取出 ArrayData
    val listSize = valueVector.getListSize()       // 2048

    require(array.numElements() == listSize)       // 维度校验
    valueVector.setNotNull(count)                  // validity bitmap 置 1

    var i = 0
    while (i < array.numElements()) {
      elementWriter.write(array, i)                // FloatWriter 逐元素写入 child vector
      i += 1
    }
  }

  def setNull(): Unit = {
    elementWriter.count += valueVector.getListSize()  // 跳过 listSize 个槽位保持对齐
    valueVector.setNull(count)                        // validity bitmap 置 0
  }
```

#### 🔑 FloatWriter（child writer）

```scala
class FloatWriter(val valueVector: Float4Vector) extends LanceArrowFieldWriter {
  def setValue(input: SpecializedGetters, ordinal: Int): Unit = {
    valueVector.setSafe(count, input.getFloat(ordinal))
  }
}
```

`setSafe` 会在底层 value buffer 的 `count * 4` 字节处写入一个 4 字节 float，并在容量不足时自动扩容。

---

### 🔗 端到端示例

以 schema `(id: Int, embedding: Array[Float] 2048 维)` 为例，完整走一遍从 writer 创建到数据落盘的路径。

<div align="center">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 380 600" width="380" height="600">
<rect width="380" height="600" rx="8" fill="#1e1e2e"/>
<rect x="40" y="20" width="300" height="130" rx="6" fill="#313244" stroke="#89b4fa" stroke-width="1.5"/>
<text x="190" y="52" text-anchor="middle" fill="#cdd6f4" font-size="18" font-weight="bold" font-family="monospace">Spark Schema</text>
<text x="60" y="82" fill="#a6adc8" font-size="15" font-family="monospace">id: IntegerType</text>
<text x="60" y="106" fill="#a6adc8" font-size="15" font-family="monospace">embedding: Array(Float)</text>
<text x="60" y="130" fill="#6c7086" font-size="13" font-family="monospace">metadata: fsl.size=2048</text>
<path d="M 190 150 L 190 200" stroke="#89b4fa" stroke-width="1.5" fill="none" marker-end="url(#b1)"/>
<text x="200" y="182" fill="#89b4fa" font-size="13" font-family="monospace">toArrowSchema()</text>
<rect x="40" y="205" width="300" height="130" rx="6" fill="#313244" stroke="#a6e3a1" stroke-width="1.5"/>
<text x="190" y="237" text-anchor="middle" fill="#cdd6f4" font-size="18" font-weight="bold" font-family="monospace">Arrow Schema</text>
<text x="60" y="267" fill="#a6adc8" font-size="15" font-family="monospace">id: Int32</text>
<text x="60" y="291" fill="#a6adc8" font-size="15" font-family="monospace">embedding: FSL(2048)</text>
<text x="60" y="315" fill="#6c7086" font-size="13" font-family="monospace">child: Float32</text>
<path d="M 190 335 L 190 385" stroke="#a6e3a1" stroke-width="1.5" fill="none" marker-end="url(#b2)"/>
<text x="200" y="367" fill="#a6e3a1" font-size="13" font-family="monospace">create(root)</text>
<rect x="40" y="390" width="300" height="145" rx="6" fill="#313244" stroke="#f9e2af" stroke-width="1.5"/>
<text x="190" y="422" text-anchor="middle" fill="#cdd6f4" font-size="18" font-weight="bold" font-family="monospace">LanceArrowWriter</text>
<text x="60" y="455" fill="#a6adc8" font-size="15" font-family="monospace">IntegerWriter</text>
<text x="60" y="482" fill="#a6adc8" font-size="15" font-family="monospace">FixedSizeListWriter</text>
<text x="78" y="509" fill="#6c7086" font-size="13" font-family="monospace">&#x2514; FloatWriter</text>
<defs>
<marker id="b1" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#89b4fa"/></marker>
<marker id="b2" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#a6e3a1"/></marker>
</defs>
</svg>
</div>

#### 📋 创建 field writer 的路径

基本逻辑是先构造 Arrow Schema，再通过 Arrow Schema 构造 field writer：

1. `SemaphoreArrowBatchWriteBuffer` 构造时，调用 `LanceArrowUtils.toArrowSchema(sparkSchema)` 将 Spark Schema 转为 Arrow Schema
2. 转换 `embedding` 字段时，`toArrowField()` 发现它是 `ArrayType(FloatType)`，并且 metadata 中包含 `arrow.fixed-size-list.size = 2048`，于是创建 `ArrowType.FixedSizeList(2048)` 类型的 Arrow Field（而非普通 `List`）
3. `VectorSchemaRoot.create(arrowSchema)` 根据 Arrow Schema 创建向量，`FixedSizeList` 类型自动生成 `FixedSizeListVector`
4. `prepareLoadNextBatch()` 调用 `LanceArrowWriter.create(root, sparkSchema)`，遍历每个 `FieldVector` 调用 `createFieldWriter()` 做模式匹配：
   - `id` → `(IntegerType, IntVector)` → `IntegerWriter`
   - `embedding` → `(ArrayType(FloatType), FixedSizeListVector)` → `FixedSizeListWriter(vector, FloatWriter)`

最终 `LanceArrowWriter` 持有两个 field writer：`[IntegerWriter, FixedSizeListWriter]`。

#### ✏️ 写入一行的过程

`arrowWriter.write(row)` 遍历 field writer 逐字段写入：

1. `IntegerWriter.write(row, 0)` → 写入 `id`
2. `FixedSizeListWriter.write(row, 1)` → 取出 `ArrayData`，校验 `numElements() == 2048`，然后循环 2048 次调用内部 `FloatWriter.write(array, i)` 逐元素写入

#### 🧠 关键点

- **维度强校验**：`numElements() == listSize`
- **null 行也要推进偏移**：null 也占 `listSize` 槽位
- **无 offset buffer**：可按 `i * listSize` 直接寻址，对向量检索场景（大量连续读取 embedding）更高效

---

### 🛠️ 我的贡献（PR #727）

[fix: write wrong offset for fixed size list with nulls](https://github.com/lance-format/lance-spark/pull/727)

我修复了 `FixedSizeListWriter` 在写入 null 行时未推进 child writer 偏移的问题。此前 `setNull()` 只标记 validity bit，导致 null 行之后数据错位。修复方式是在 null 分支执行 `elementWriter.count += listSize`，并补充了 null 交错和嵌套场景测试。🎯