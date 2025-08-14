---
tags: [spark, gluten, velox, iceberg]
---
## Write Strategy for Iceberg Bucket Tables in Gluten

### Motivation for This Design

We have designed a native write solution for Iceberg tables in [Gluten](https://gluten.apache.org/) (based on [Velox](https://velox-lib.io/)).
When using the Gluten compute engine, data is stored in a columnar format, and we need to write this data to HDFS natively to avoid falling back to the JVM.

Velox’s Parquet writer natively supports columnar writes. However, in many of our business scenarios, Iceberg tables are bucketed tables, which play a crucial role in [SPJ](https://issues.apache.org/jira/browse/SPARK-37375).
For bucketed tables, the main challenge with columnar writes is determining the correct directory for each columnar batch. If the directory is incorrect, it will affect the correctness of data reads.

### Current Spark-Iceberg Write Strategy for Bucket Tables

When writing to bucketed tables in Iceberg, if the distribution mode is set to hash, Spark first shuffles data by the partition key and then sorts it.
This way, each task only needs to open one writer per partition directory (ignoring file rolling).

### How to Write by Bucket ID

To ensure that each columnar batch is written to the correct directory, we implement the following steps:

1. On the Spark side, support shuffling by bucket ID.
2. Ensuring that all data in a partition is written to the same bucket.
3. Calculate the target directory for writing.
4. Use the Velox writer to perform writing.

![shuffle by bucket id](/assets/images/spark-shuffle-by-bucket-id.png)

#### Using Bucket ID as Partition ID

The Spark expression for calculating the partition ID is:

```
Pmod(new Murmur3Hash(expressions), Literal(numPartitions))
```

To partition by bucket ID, we need to modify the original expression and ensure that the number of partitions equals the number of buckets.
This can be achieved by updating the relevant rules. With this change, all data in a partition will be written to a single bucket.

#### Calculating the Target Directory

Fetching a single row from a columnar batch is very efficient and does not require converting the entire batch to row format.
By extracting just one row, we can determine the target directory for writing, and then use the Velox writer to complete the write operation.

#### Preventing Partition Coalescing from Breaking the Logic

We set the number of shuffle partitions to match the number of buckets.
However, Adaptive Query Execution (AQE) may perform [Partition Coalescing](https://spark.apache.org/docs/3.5.3/sql-performance-tuning.html#coalescing-post-shuffle-partitions), which changes the number of partitions. 
To prevent this, we also need to modify the rules for Partition Coalescing.
