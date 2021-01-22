# ExecutorKubernetesCredentialsFeatureStep

`ExecutorKubernetesCredentialsFeatureStep` is...FIXME

## <span id="executorServiceAccount"> spark.kubernetes.authenticate.executor.serviceAccountName

`ExecutorKubernetesCredentialsFeatureStep` uses [spark.kubernetes.authenticate.executor.serviceAccountName](configuration-properties.md#spark.kubernetes.authenticate.executor.serviceAccountName) configuration property when requested to [configure a pod](#configurePod).

## <span id="configurePod"> Configuring Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod` [buildPodWithServiceAccount](KubernetesUtils.md#buildPodWithServiceAccount) when...FIXME
