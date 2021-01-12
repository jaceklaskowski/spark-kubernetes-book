# BasicExecutorFeatureStep

`BasicExecutorFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`BasicExecutorFeatureStep` takes the following to be created:

* <span id="kubernetesConf"> [KubernetesExecutorConf](KubernetesExecutorConf.md)
* <span id="secMgr"> `SecurityManager` (Apache Spark)

`BasicExecutorFeatureStep` is created when:

* `KubernetesExecutorBuilder` is requested to [build a pod spec for executors](KubernetesExecutorBuilder.md#buildFromFeatures)

## <span id="executorContainerImage"> Executor Container Image Name

`BasicExecutorFeatureStep` asserts that [spark.kubernetes.executor.container.image](configuration-properties.md#spark.kubernetes.executor.container.image) configuration property is defined or throws a `SparkException`:

```text
Must specify the executor container image
```

## <span id="configurePod"> Configuring Driver Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod`...FIXME
