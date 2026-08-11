---
tags: [AI, spark, lance, arrow]
lang: en
ref: spark-df-write-to-lance
---

## ✨ How a Spark DataFrame Is Written to Lance via `lance-spark`

### 👋 Background

Spark introduced **DataSource V2** in 2.3, defining a standard contract for external sources (`Table`, `ScanBuilder`, `WriteBuilder`, etc.). Compared with V1, V2 provides a cleaner write protocol and clearer two-phase commit semantics.

With `lance-spark`, writing is driven by this protocol. When users run:

```scala
df.write.format("lance").save(path)
```

Spark executes the V2 write lifecycle and persists data in Lance's columnar format. 🚀

---

### 🔧 Capabilities Provided by Spark

- Declare capabilities: `SupportsWrite` / `SupportsTruncate`
- Build write: `newWriteBuilder(...)` → `BatchWrite`
- Parallel execution: Driver distributes `DataWriterFactory`, Executor calls `createWriter(...)`
- Two-phase commit: Executor returns `WriterCommitMessage`, Driver calls `commit(messages)`; on failure `abort(...)`

---

### 🧭 High-level Write Flow (DataSource V2)

1. `df.write.format("lance").save(path)`
2. Build `WriteBuilder` / `BatchWrite`
3. Driver creates and distributes `DataWriterFactory`
4. Executors run `createWriter(...)` and write in parallel
5. Executors return `WriterCommitMessage`
6. Driver performs `commit(messages)` (or `abort(...)` on failure)

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
<text x="60" y="133" fill="#a6adc8" font-size="14" font-family="monospace">- build WriteBuilder / BatchWrite</text>
<text x="60" y="158" fill="#a6adc8" font-size="14" font-family="monospace">- createBatchWriterFactory</text>
<line x1="180" y1="200" x2="448" y2="200" stroke="#cdd6f4" stroke-width="1.5"/>
<polygon points="448,196 458,200 448,204" fill="#cdd6f4"/>
<text x="240" y="192" fill="#cdd6f4" font-size="13" font-family="monospace">send DataWriterFactory</text>
<rect x="360" y="225" width="200" height="70" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="1"/>
<text x="380" y="253" fill="#a6adc8" font-size="14" font-family="monospace">- createWriter</text>
<text x="380" y="278" fill="#a6adc8" font-size="14" font-family="monospace">- write in parallel</text>
<line x1="455" y1="325" x2="202" y2="325" stroke="#cdd6f4" stroke-width="1.5" stroke-dasharray="6,4"/>
<polygon points="202,320 192,325 202,330" fill="#cdd6f4"/>
<text x="220" y="317" fill="#cdd6f4" font-size="13" font-family="monospace">return WriterCommitMessage</text>
<rect x="120" y="350" width="120" height="45" rx="5" fill="#313244" stroke="#f9e2af" stroke-width="1"/>
<text x="140" y="378" fill="#a6adc8" font-size="14" font-family="monospace">- commit</text>
</svg>
</div>

---

### 🧩 Spark Schema vs Lance (Arrow) Schema

Spark uses `StructType`, while Lance uses Arrow Schema. Primitive types (`Int`, `Long`, `Float`, `String`, etc.) map naturally, but Arrow carries richer type semantics.

- **FixedSizeList vs ArrayType**: Arrow distinguishes fixed-size and variable-size lists, while Spark only has `ArrayType`.
- **LargeUtf8 / LargeBinary**: Arrow supports 64-bit-offset large object types.
- **Float16**: Arrow supports half precision; Spark has no native equivalent.

In `lance-spark`, these extra semantics are preserved through field metadata (for example, `arrow.fixed-size-list.size`) and restored by `LanceArrowUtils.toArrowSchema()`. ✅

---

### 📝 Lance Writer Internals

`LanceArrowWriter` holds an array of field writers, one per column:

```scala
class LanceArrowWriter(root: VectorSchemaRoot, fields: Array[LanceArrowFieldWriter])
```

On the Executor side, Spark passes rows as `InternalRow`, but Lance stores data in Arrow columnar format. So for each row received, `LanceArrowWriter` iterates all field writers and each appends one element to its column vector — performing the **row-to-column** transformation.

---

### ⚙️ Field Writer Detail: `FixedSizeListWriter`

#### 🗂️ Arrow FixedSizeList Memory Layout

Per the [Arrow Columnar Format](https://arrow.apache.org/docs/format/Columnar.html#fixed-size-list-layout), a `FixedSizeList<Float32>[2048]` vector consists of two parts:

```
FixedSizeListVector (listSize=2048)
├── validity buffer: 1 bit per row, marks null/non-null
└── child Float4Vector (value buffer): contiguous storage of all rows' elements
    → row0's 2048 floats | row1's 2048 floats | ...
```

No offset buffer — since each row has a fixed element count, row i starts at position `i * listSize`.

#### 📖 FixedSizeListWriter Source Logic

```scala
class FixedSizeListWriter(
    val valueVector: FixedSizeListVector,
    val elementWriter: LanceArrowFieldWriter)  // FloatWriter here

  def setValue(input: SpecializedGetters, ordinal: Int): Unit = {
    val array = input.getArray(ordinal)            // get ArrayData from Spark Row
    val listSize = valueVector.getListSize()       // 2048

    require(array.numElements() == listSize)       // dimension check
    valueVector.setNotNull(count)                  // set validity bitmap to 1

    var i = 0
    while (i < array.numElements()) {
      elementWriter.write(array, i)                // FloatWriter writes to child vector
      i += 1
    }
  }

  def setNull(): Unit = {
    elementWriter.count += valueVector.getListSize()  // skip listSize slots to maintain alignment
    valueVector.setNull(count)                        // set validity bitmap to 0
  }
```

#### 🔑 FloatWriter (Child Writer)

```scala
class FloatWriter(val valueVector: Float4Vector) extends LanceArrowFieldWriter {
  def setValue(input: SpecializedGetters, ordinal: Int): Unit = {
    valueVector.setSafe(count, input.getFloat(ordinal))
  }
}
```

`setSafe` writes a 4-byte float at offset `count * 4` in the underlying value buffer, auto-expanding capacity when needed.

---

### 🔗 End-to-End Example

For schema `(id: Int, embedding: Array[Float] with 2048 dims)`, let's trace the full path from writer creation to data written to disk.

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

#### 📋 Field Writer Creation Path

The basic logic is: first construct the Arrow Schema, then create field writers from it:

1. When `SemaphoreArrowBatchWriteBuffer` is constructed, it calls `LanceArrowUtils.toArrowSchema(sparkSchema)` to convert Spark Schema to Arrow Schema
2. When converting the `embedding` field, `toArrowField()` sees it is `ArrayType(FloatType)` with metadata `arrow.fixed-size-list.size = 2048`, so it creates an `ArrowType.FixedSizeList(2048)` Arrow Field (not a regular `List`)
3. `VectorSchemaRoot.create(arrowSchema)` creates vectors from the Arrow Schema — `FixedSizeList` type automatically produces a `FixedSizeListVector`
4. `prepareLoadNextBatch()` calls `LanceArrowWriter.create(root, sparkSchema)`, iterating each `FieldVector` and calling `createFieldWriter()` for pattern matching:
   - `id` → `(IntegerType, IntVector)` → `IntegerWriter`
   - `embedding` → `(ArrayType(FloatType), FixedSizeListVector)` → `FixedSizeListWriter(vector, FloatWriter)`

Finally, `LanceArrowWriter` holds two field writers: `[IntegerWriter, FixedSizeListWriter]`.

#### ✏️ Writing a Single Row

`arrowWriter.write(row)` iterates field writers and writes each field:

1. `IntegerWriter.write(row, 0)` → writes `id`
2. `FixedSizeListWriter.write(row, 1)` → extracts `ArrayData`, validates `numElements() == 2048`, then loops 2048 times calling `FloatWriter.write(array, i)` element-by-element

#### 🧠 Key Takeaways

- **Strong dimension check**: `numElements() == listSize`
- **Null rows still move offsets**: null rows also consume `listSize` slots
- **No offset buffer**: direct addressing via `i * listSize`, more efficient for vector search scenarios (continuous embedding reads)

---

### 🛠️ My Contribution (PR #727)

[fix: write wrong offset for fixed size list with nulls](https://github.com/lance-format/lance-spark/pull/727)

I fixed a null-handling offset bug in `FixedSizeListWriter`. Previously, null rows did not advance child writer offsets, which misaligned all subsequent values. The fix advances by `listSize` in the null branch and adds tests for interleaved nulls and nested cases. 🎯