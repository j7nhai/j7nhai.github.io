---
mermaid: true
---
## Some Debug Skills when developing Gluten-Velox

### Log Velox Plan

可以通过 spark conf 来控制 Velox Log 的级别, 对它们对解释可以在 Gluten 网站找到。

```
spark.gluten.sql.substrait.plan.logLevel INFO
spark.gluten.sql.injectNativePlanStringToExplain true
spark.gluten.sql.columnar.backend.velox.showTaskMetricsWhenFinished true

spark.gluten.sql.columnar.backend.velox.glogVerboseLevel 0
spark.gluten.sql.columnar.backend.velox.glogSeverityLevel 0
```

### Analysis Core File

要分析 core 文件， 首先需要接触 core 文件大小限制。

```
ulimit -c unlimited
```

其次， 如果你的程序运行在 kubernetes 集群上， 还需要开启特权。

### 本地 demo 输出日志

在本地执行 Velox， 可以通过如下方法打开日志。 

```c++
  FLAGS_v = 0;
  FLAGS_minloglevel = 0;
  FLAGS_logtostderr = true;
```

### 打印调用栈

```c++
LOG(WARNING) << process::StackTrace().toString();
```

### Java 端只测试某个特定的类

```
mvn test -DwildcardSuites=org.apache.XxxYy -Pxxx
```
