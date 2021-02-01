# KubernetesExecutorConf

`KubernetesExecutorConf` is a [KubernetesConf](KubernetesConf.md) (for [KubernetesExecutorBuilder](KubernetesExecutorBuilder.md) to [build a pod spec for executors](KubernetesExecutorBuilder.md#buildFromFeatures)).

![KubernetesExecutorConf, ExecutorPodsAllocator and KubernetesExecutorBuilder](images/KubernetesExecutorConf.png)

## Creating Instance

`KubernetesExecutorConf` takes the following to be created:

* <span id="sparkConf"> `SparkConf`
* <span id="appId"> Application ID
* <span id="executorId"> Executor ID
* <span id="driverPod"> Driver Pod

`KubernetesExecutorConf` is created when:

* `ExecutorPodsAllocator` is requested to [handle executor pods snapshots](ExecutorPodsAllocator.md#onNewSnapshots) (and requests missing executors from Kubernetes via [KubernetesConf utility](KubernetesConf.md#createExecutorConf))

## <span id="volumes"> Volume Specs

```scala
volumes: Seq[KubernetesVolumeSpec]
```

`volumes` is part of the [KubernetesConf](KubernetesConf.md#volumes) abstraction.

`volumes` [parses volume specs](KubernetesVolumeUtils.md#parseVolumesWithPrefix) for the executor pod (with the **spark.kubernetes.executor.volumes.** prefix) from the [SparkConf](#sparkConf).
