# KubernetesExecutorConf

`KubernetesExecutorConf` is a [KubernetesConf](KubernetesConf.md) (for [KubernetesExecutorBuilder](KubernetesExecutorBuilder.md) to [build a pod spec for executors](KubernetesExecutorBuilder.md#buildFromFeatures)).

![KubernetesExecutorConf, ExecutorPodsAllocator and KubernetesExecutorBuilder](images/KubernetesExecutorConf.png)

## Creating Instance

`KubernetesExecutorConf` takes the following to be created:

* <span id="sparkConf"> `SparkConf` ([Apache Spark]({{ book.spark_core }}/SparkConf))
* <span id="appId"> Application ID
* <span id="executorId"> Executor ID
* <span id="driverPod"> Driver Pod
* <span id="resourceProfileId"> Resource Profile ID (default: `0`)

`KubernetesExecutorConf` is created when:

* `ExecutorPodsAllocator` is requested for [new executors](ExecutorPodsAllocator.md#requestNewExecutors) (and [createExecutorConf](KubernetesConf.md#createExecutorConf))

## <span id="volumes"> Volumes

```scala
volumes: Seq[KubernetesVolumeSpec]
```

`volumes` [parses volume specs](KubernetesVolumeUtils.md#parseVolumesWithPrefix) for the executor pod (with the **spark.kubernetes.executor.volumes.** prefix) from the [SparkConf](#sparkConf).

`volumes` is part of the [KubernetesConf](KubernetesConf.md#volumes) abstraction.

## <span id="resourceNamePrefix"> Resource Name Prefix

```scala
resourceNamePrefix: String
```

`resourceNamePrefix` is the value of the [spark.kubernetes.executor.podNamePrefix](configuration-properties.md#spark.kubernetes.executor.podNamePrefix) (if defined) or the [resource name prefix](KubernetesConf.md#getResourceNamePrefix) (for the [appName](KubernetesConf.md#appName)).

`resourceNamePrefix` is part of the [KubernetesConf](KubernetesConf.md#resourceNamePrefix) abstraction.
