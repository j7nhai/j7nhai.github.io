---
tags: [spark, gluten, velox, iceberg]
lang: zh
ref: spark-columnar-shuffle-write-by-bucket-id
permalink: /zh/2025/03/13/spark-columnar-shuffle-write-by-bucket-id.html
---

## 🧊🪣 Gluten 中 Iceberg Bucket 表的写入策略

### 🎯 设计动机

我们在 [Gluten](https://gluten.apache.org/)（基于 [Velox](https://velox-lib.io/)）中为 Iceberg 表设计了一套原生写入方案。
使用 Gluten 计算引擎时，数据以列式格式存储；为了避免回退到 JVM，我们希望能够把这份列式数据**直接原生写入**到 HDFS。

Velox 的 Parquet writer 原生支持列式写入。但在很多业务场景里，Iceberg 表是 **bucketed table（分桶表）**，它在 [SPJ](https://issues.apache.org/jira/browse/SPARK-37375) 等优化中扮演着非常关键的角色。
对于分桶表，列式写入的核心挑战在于：**每个 columnar batch 应该写到哪个目录**。目录选错会直接影响读的正确性。

### 🧭 当前 Spark-Iceberg 对分桶表的写入策略

当向 Iceberg 的分桶表写入时，如果 distribution mode 设置为 hash，Spark 会先按 partition key 对数据做 shuffle，然后再做 sort。
这样每个 task 只需要为每个 partition 目录打开一个 writer（忽略文件滚动 file rolling）。

### 🧮🪣 如何按 Bucket ID 写入

为了保证每个 columnar batch 被写到正确的目录，我们做了如下步骤：

1. 🧩 在 Spark 侧支持按 bucket ID 进行 shuffle。
2. 🧷 确保同一个 partition 内的数据只会写入同一个 bucket。
3. 🧭 计算写入的目标目录。
4. ✍️ 调用 Velox writer 完成实际写入。

![shuffle by bucket id](/assets/images/spark-shuffle-by-bucket-id.png)

#### 🧮 使用 Bucket ID 作为 Partition ID

Spark 用于计算 partition ID 的表达式是：

```
Pmod(new Murmur3Hash(expressions), Literal(numPartitions))
```

要按 bucket ID 进行分区，我们需要修改原表达式，并确保 partition 数量等于 bucket 数量。
这可以通过更新相关的 rules 来实现。完成后，同一个 partition 内的数据就会写到同一个 bucket。

#### 📁 计算目标目录

从一个 columnar batch 里取出单行的开销非常低，并不需要把整个 batch 转成 row 格式。
我们只需要抽取一行，就能确定写入的目标目录，然后交给 Velox writer 完成写入。

#### 🛡️ 防止 Partition Coalescing 破坏逻辑

我们把 shuffle 分区数设置成与 bucket 数相同。
但是 AQE（Adaptive Query Execution）可能会做 [Partition Coalescing](https://spark.apache.org/docs/3.5.3/sql-performance-tuning.html#coalescing-post-shuffle-partitions)，从而改变分区数。
为避免这种情况，我们也需要修改 Partition Coalescing 的相关规则，防止其影响 bucket 写入逻辑。
