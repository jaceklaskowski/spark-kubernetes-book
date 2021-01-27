# DriverKubernetesCredentialsFeatureStep

`DriverKubernetesCredentialsFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`DriverKubernetesCredentialsFeatureStep` takes the following to be created:

* <span id="kubernetesConf"> [KubernetesConf](KubernetesConf.md)

`DriverKubernetesCredentialsFeatureStep` is created when:

* `KubernetesDriverBuilder` is requested for the [driver pod spec](KubernetesDriverBuilder.md#buildFromFeatures)

## <span id="configurePod"> Configuring Driver Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod`...FIXME
