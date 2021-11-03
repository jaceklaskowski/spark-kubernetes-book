# BasicExecutorFeatureStep

`BasicExecutorFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`BasicExecutorFeatureStep` takes the following to be created:

* <span id="kubernetesConf"> [KubernetesExecutorConf](KubernetesExecutorConf.md)
* <span id="secMgr"> `SecurityManager` (Apache Spark)
* <span id="resourceProfile"> `ResourceProfile` ([Apache Spark]({{ book.spark_core }}/stage-level-scheduling/ResourceProfile))

`BasicExecutorFeatureStep` is created when:

* `KubernetesExecutorBuilder` is requested for a [pod spec for executors](KubernetesExecutorBuilder.md#buildFromFeatures)

## <span id="execResources"> Executor Resources

`BasicExecutorFeatureStep` creates an `ExecutorResourcesOrDefaults` when [created](#creating-instance) (cf. [ResourceProfile]({{ book.spark_core }}/stage-level-scheduling/ResourceProfile#getResourcesForClusterManager)) with the following:

* ID and `executorResources` of the [ResourceProfile](#resourceProfile)
* [spark.kubernetes.memoryOverheadFactor](configuration-properties.md#spark.kubernetes.memoryOverheadFactor) as the `overheadFactor`

The `ExecutorResourcesOrDefaults` is used for the following:

* [executorMemoryString](#executorMemoryString)
* [executorCoresRequest](#executorCoresRequest)
* [Configuring Pod](#configurePod) (for `executorMemoryQuantity`, `executorResourceQuantities` and `SPARK_EXECUTOR_CORES` environment variable)

## <span id="executorContainerImage"> Executor Container Image Name

`BasicExecutorFeatureStep` asserts that [spark.kubernetes.executor.container.image](configuration-properties.md#spark.kubernetes.executor.container.image) configuration property is defined or throws a `SparkException`:

```text
Must specify the executor container image
```

## <span id="executorPodNamePrefix"> Executor Pod Name Prefix

`BasicExecutorFeatureStep` requests the [KubernetesExecutorConf](#kubernetesConf) for the [resourceNamePrefix](KubernetesExecutorConf.md#resourceNamePrefix).

## <span id="configurePod"> Configuring Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` builds the name of the pod (with the [executorPodNamePrefix](#executorPodNamePrefix) and the [executorId](KubernetesExecutorConf.md#executorId) of the [KubernetesExecutorConf](#kubernetesConf)):

```text
[executorPodNamePrefix]-exec-[executorId]
```

`configurePod` configures a `PodBuilder` and a `ContainerBuilder`, and in the end, creates a `SparkPod`.

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

### <span id="buildExecutorResourcesQuantities"> buildExecutorResourcesQuantities

```scala
buildExecutorResourcesQuantities(
  customResources: Set[ExecutorResourceRequest]): Map[String, Quantity]
```

For every `ExecutorResourceRequest` (in the given `customResources`), `buildExecutorResourcesQuantities` [builds a Kubernetes resource name](KubernetesConf.md#buildKubernetesResourceName) (based on the `vendor` and the `resourceName` of the `ExecutorResourceRequest`) and maps it to the `amount` (of the `ExecutorResourceRequest`).
