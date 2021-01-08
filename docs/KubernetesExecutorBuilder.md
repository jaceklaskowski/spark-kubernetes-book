# KubernetesExecutorBuilder

## Creating Instance

`KubernetesExecutorBuilder` takes no arguments to be created (_and could really be a Scala utility object_).

`KubernetesExecutorBuilder` is created when:

* `KubernetesClusterManager` is requested to [create a SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend) (and creates a [ExecutorPodsAllocator](ExecutorPodsAllocator.md))

## <span id="buildFromFeatures"> Building Executor Pod Specification

```scala
buildFromFeatures(
  conf: KubernetesExecutorConf,
  secMgr: SecurityManager,
  client: KubernetesClient): SparkPod
```

When defined, `buildFromFeatures` [loads the pod spec](KubernetesUtils.md#loadPodFromTemplate) from the pod template file (based on the [spark.kubernetes.executor.podTemplateFile](configuration-properties.md#spark.kubernetes.executor.podTemplateFile) and [spark.kubernetes.executor.podTemplateContainerName](configuration-properties.md#spark.kubernetes.executor.podTemplateContainerName) configuration properties). Otherwise, `buildFromFeatures` starts from an initial empty pod specification.

In the end, `buildFromFeatures` [configures the executor pod specification](KubernetesFeatureConfigStep.md#configurePod) through a series of the feature steps:

* [BasicExecutorFeatureStep](BasicExecutorFeatureStep.md)
* [ExecutorKubernetesCredentialsFeatureStep](ExecutorKubernetesCredentialsFeatureStep.md)
* [MountSecretsFeatureStep](MountSecretsFeatureStep.md)
* [EnvSecretsFeatureStep](EnvSecretsFeatureStep.md)
* [MountVolumesFeatureStep](MountVolumesFeatureStep.md)
* [LocalDirsFeatureStep](LocalDirsFeatureStep.md)

`buildFromFeatures` is used when:

* `ExecutorPodsAllocator` is requested to [handle executor pods snapshots](ExecutorPodsAllocator.md#onNewSnapshots) (and requests missing executors from Kubernetes)
