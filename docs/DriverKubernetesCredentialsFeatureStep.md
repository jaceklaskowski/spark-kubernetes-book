# DriverKubernetesCredentialsFeatureStep

`DriverKubernetesCredentialsFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md) for [KubernetesDriverBuilder](KubernetesDriverBuilder.md) (to [build a driver pod spec](KubernetesDriverBuilder.md#buildFromFeatures)).

## Creating Instance

`DriverKubernetesCredentialsFeatureStep` takes the following to be created:

* <span id="kubernetesConf"> [KubernetesConf](KubernetesConf.md)

`DriverKubernetesCredentialsFeatureStep` is created when:

* `KubernetesDriverBuilder` is requested for the [driver pod spec](KubernetesDriverBuilder.md#buildFromFeatures)

## <span id="driverServiceAccount"> spark.kubernetes.authenticate.driver.serviceAccountName

`DriverKubernetesCredentialsFeatureStep` uses the [spark.kubernetes.authenticate.driver.serviceAccountName](configuration-properties.md#spark.kubernetes.authenticate.driver.serviceAccountName) configuration property for [configuring a pod](#configurePod).

## <span id="configurePod"> Configuring Driver Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod`...FIXME
