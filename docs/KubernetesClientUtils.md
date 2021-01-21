# KubernetesClientUtils

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
