---
tags: [spark, parquet, velox, iceberg]
lang: zh
ref: parquet-field-id
permalink: /zh/2025/07/31/parquet-field-id.html
---

## 🧩🧱 数据湖场景下 Parquet Field ID 的一些记录

### 👋 背景

从 Spark 3.3 开始，对于带 schema ID 的 schema，写入 Parquet 时会同时把 **field ID** 写进 Parquet 文件。
这么做的主要原因，是为了兼容 Iceberg、Delta Lake 等依赖 field ID 做 schema 管理的数据湖格式。
有了 field ID 之后，即使新增/删除字段，老数据也能被正确读取。

```
Parquet File
└── Footer (FileMetaData)
    └── schema: List<SchemaElement>
        ├── SchemaElement (root)
        ├── SchemaElement (field1)
        │     ├── name: "id"
        │     ├── type: INT64
        │     └── field_id: 1   <---
        ├── SchemaElement (field2)
        │     ├── name: "name"
        │     ├── type: STRING
        │     └── field_id: 2   <---
        └── ...
```

📚 Iceberg 官网规范里也提到了 field ID，见 [This](https://iceberg.apache.org/spec/#schemas)。

### ✍️📦 支持写入 Field ID 的 Parquet Writer

#### 🏹 Apache Arrow

很多框架的 Parquet writer 都支持设置 field ID，例如 Parquet Java API。
而在 Arrow C++ 里，这个接口看起来更“绕”一些：它是通过 `key_value_metadata` 来设置的。

```cpp
auto name_field = arrow::field(
    "name", arrow::utf8(),
    /*nullable=*/false,
    arrow::key_value_metadata({"PARQUET:field_id"}, {"2"})
);

...

auto schema = arrow::schema(
    {id_field, name_field, score_field});
    
...

auto table = arrow::Table::Make(
    schema, {id_array, name_array, score_array});
```

Reference: [This](https://github.com/apache/arrow/blob/release-15.0.0-rc1/cpp/src/parquet/arrow/writer.h#L51)

#### 🦊 Facebook Velox

Velox 的 Parquet writer 基于 Arrow，因此理论上也可以支持 field ID，但目前还没有实现。

### 🔎📖 支持查看 Field ID 的 Parquet Reader

#### ☕️ Apache Parquet Java

支持。

```scala
val schema: MessageType = footer
	.getFileMetaData
  .getSchema

schema.getFields.asScala.foreach {
  field =>
  	field.getId
  	// ...
}
```

#### 🧰 parquet-tools

示例：

```bash
parquet-tools schema xxx.parquet --format raw | json_pp
```

### 🧊🧷 Iceberg 中 Parquet 文件里的 Field ID

它们对应的是 schema 里的 ID，所以 Parquet 文件里列的物理存储顺序并不重要。
Iceberg 会基于 schema 的 ID 去文件里找到对应的列。

例如在初始化 Arrow 的列式 reader 时，会同时使用 Iceberg schema 与 Parquet schema。
在 visit 过程中，它们会通过 field ID 关联起来。

```java
  public static ColumnarBatchReader buildReader(
      Schema expectedSchema,
      MessageType fileSchema,
      Map<Integer, ?> idToConstant,
      DeleteFilter<InternalRow> deleteFilter) {
    return (ColumnarBatchReader)
        TypeWithSchemaVisitor.visit(
            expectedSchema.asStruct(),
            fileSchema,
            new ReaderBuilder(
                expectedSchema,
                fileSchema,
                NullCheckingForGet.NULL_CHECKING_ENABLED,
                idToConstant,
                ColumnarBatchReader::new,
                deleteFilter));
  }
```
