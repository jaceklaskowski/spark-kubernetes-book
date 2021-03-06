# KubernetesDriverBuilder

`KubernetesDriverBuilder` is used to [build a specification of a driver pod](#buildFromFeatures) (for a Spark application deployed in [cluster](overview.md#cluster-deploy-mode) deploy mode).

![KubernetesDriverBuilder, Client and KubernetesClientApplication](images/KubernetesDriverBuilder.png)

## Creating Instance

`KubernetesDriverBuilder` takes no arguments to be created.

`KubernetesDriverBuilder` is created when:

* `KubernetesClientApplication` is requested to [start](KubernetesClientApplication.md#start)

## <span id="KubernetesDriverSpec"> KubernetesDriverSpec

`KubernetesDriverSpec` is the following:

* <span id="pod"> `SparkPod`
* <span id="driverKubernetesResources"> Driver Resources
* <span id="systemProperties"> System Properties

## <span id="buildFromFeatures"> Building Pod Spec for Driver

```scala
buildFromFeatures(
  conf: KubernetesDriverConf,
  client: KubernetesClient): KubernetesDriverSpec
```

`buildFromFeatures` creates an initial driver pod specification.

With [spark.kubernetes.driver.podTemplateFile](configuration-properties.md#spark.kubernetes.driver.podTemplateFile) configuration property defined, `buildFromFeatures` [loads it](KubernetesUtils.md#loadPodFromTemplate) (with the given `KubernetesClient` and the container name based on [spark.kubernetes.driver.podTemplateContainerName](configuration-properties.md#spark.kubernetes.driver.podTemplateContainerName) configuration property) or defaults to an empty pod specification.

`buildFromFeatures` builds a [KubernetesDriverSpec](#KubernetesDriverSpec) (with the initial driver pod specification).

In the end, `buildFromFeatures` [configures the driver pod specification](KubernetesFeatureConfigStep.md#configurePod) (with [additional system properties](KubernetesFeatureConfigStep.md#getAdditionalPodSystemProperties) and [additional resources](KubernetesFeatureConfigStep.md#getAdditionalKubernetesResources)) through a series of the feature steps:

* [BasicDriverFeatureStep](BasicDriverFeatureStep.md)
* [DriverKubernetesCredentialsFeatureStep](DriverKubernetesCredentialsFeatureStep.md)
* [DriverServiceFeatureStep](DriverServiceFeatureStep.md)
* [MountSecretsFeatureStep](MountSecretsFeatureStep.md)
* [EnvSecretsFeatureStep](EnvSecretsFeatureStep.md)
* [MountVolumesFeatureStep](MountVolumesFeatureStep.md)
* [DriverCommandFeatureStep](DriverCommandFeatureStep.md)
* HadoopConfDriverFeatureStep
* KerberosConfDriverFeatureStep
* [PodTemplateConfigMapStep](PodTemplateConfigMapStep.md)
* [LocalDirsFeatureStep](LocalDirsFeatureStep.md)

`buildFromFeatures` is used when:

* `Client` is requested to [run](Client.md#run)
