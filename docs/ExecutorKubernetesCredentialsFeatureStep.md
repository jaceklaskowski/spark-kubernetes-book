# ExecutorKubernetesCredentialsFeatureStep

`ExecutorKubernetesCredentialsFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md) for [KubernetesExecutorBuilder](KubernetesExecutorBuilder.md) (to [build a pod spec for executors](KubernetesExecutorBuilder.md#buildFromFeatures)).

## Creating Instance

`ExecutorKubernetesCredentialsFeatureStep` takes the following to be created:

* <span id="kubernetesConf"> [KubernetesConf](KubernetesConf.md)

`ExecutorKubernetesCredentialsFeatureStep` is created when:

* `KubernetesExecutorBuilder` is requested for a [pod spec for executors](KubernetesExecutorBuilder.md#buildFromFeatures)

## <span id="driverServiceAccount"> spark.kubernetes.authenticate.driver.serviceAccountName

`ExecutorKubernetesCredentialsFeatureStep` uses [spark.kubernetes.authenticate.executor.serviceAccountName](configuration-properties.md#spark.kubernetes.authenticate.executor.serviceAccountName) configuration property when requested to [configure a pod](#configurePod).

## <span id="executorServiceAccount"> spark.kubernetes.authenticate.executor.serviceAccountName

`ExecutorKubernetesCredentialsFeatureStep` uses [spark.kubernetes.authenticate.executor.serviceAccountName](configuration-properties.md#spark.kubernetes.authenticate.executor.serviceAccountName) configuration property when requested to [configure a pod](#configurePod).

## <span id="configurePod"> Configuring Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod` [buildPodWithServiceAccount](KubernetesUtils.md#buildPodWithServiceAccount) when...FIXME
