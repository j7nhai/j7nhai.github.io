## Something about Parquet Field IDs in Data Lakes

### Introduction

Starting from Spark 3.3, for schemas with schema IDs, a field ID will also be written into the Parquet file during write operations. The main reason is to be compatible with data lake formats such as Iceberg and Delta Lake, which rely on field IDs for schema management. With field IDs, when adding or removing fields, old data can still be read correctly.

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

Information about field IDs is also provided in iceberg official website. See: https://iceberg.apache.org/spec/#schemas

### Parquet Writers That Support Writing Field IDs

#### Apache Arrow
Many frameworks' Parquet writers can set field IDs during write, such as the Parquet Java API. In Arrow C++, the interface appears more cumbersome. It is set via special key_value_metadata.

```c++
auto name_field = arrow::field(
    "name", arrow::utf8(),
    /*nullable=*/false,
    arrow::key_value_metadata({"PARQUET:field_id"}, {"2"})
);
```

Reference: https://github.com/apache/arrow/blob/release-15.0.0-rc1/cpp/src/parquet/arrow/writer.h#L51

#### Facebook Velox

Velox's Parquet writer is based on Arrow, so in theory it can support field IDs, but currently it is not supported.

### Parquet Readers That Support Viewing Field IDs

#### Apache Parquet Java

Supported

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

#### Parquet-tools
Not yet supported

### Field IDs in Parquet Files in Iceberg

They correspond to the IDs in the schema, so the storage order of columns in the Parquet file does not matter. Iceberg will use the IDs in the schema to find the corresponding columns in the file.

For example, when initializing Arrow's columnar reader, both the Iceberg schema and the Parquet schema are used for initialization. In the visit function, they are linked together via the field ID.

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

