# KubernetesClientUtils Utility

## <span id="configMapNameExecutor"> Name of Config Map for Executors

```scala
configMapNameExecutor: String
```

`KubernetesClientUtils` defines the name of a config map for executors as follows:

```text
spark-exec-[uniqueID]-conf-map
```

`spark-exec-[uniqueID]` is made under 54 characters to let `configMapNameExecutor` be at 63 characters maximum.

## <span id="buildSparkConfDirFilesMap"> buildSparkConfDirFilesMap

```scala
buildSparkConfDirFilesMap(
  configMapName: String,
  sparkConf: SparkConf,
  resolvedPropertiesMap: Map[String, String]): Map[String, String]
```

`buildSparkConfDirFilesMap`...FIXME

`buildSparkConfDirFilesMap` is used when:

* `BasicExecutorFeatureStep` is requested to [configure a pod](BasicExecutorFeatureStep.md#configurePod)
* `Client` is requested to [run](Client.md#run)
* `KubernetesClusterSchedulerBackend` is requested to [setUpExecutorConfigMap](KubernetesClusterSchedulerBackend.md#setUpExecutorConfigMap)

### <span id="loadSparkConfDirFiles"> loadSparkConfDirFiles

```scala
loadSparkConfDirFiles(
  conf: SparkConf): Map[String, String]
```

`loadSparkConfDirFiles`...FIXME
