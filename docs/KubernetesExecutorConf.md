# KubernetesExecutorConf

`KubernetesExecutorConf` is a [KubernetesConf](KubernetesConf.md).

## Creating Instance

`KubernetesExecutorConf` takes the following to be created:

* <span id="sparkConf"> `SparkConf`
* <span id="appId"> Application ID
* <span id="executorId"> Executor ID
* <span id="driverPod"> Optional Driver Pod

`KubernetesExecutorConf` is created when:

* `ExecutorPodsAllocator` is requested to [handle executor pods snapshots](ExecutorPodsAllocator.md#onNewSnapshots) (and requests missing executors from Kubernetes via [KubernetesConf utility](KubernetesConf.md#createExecutorConf))
