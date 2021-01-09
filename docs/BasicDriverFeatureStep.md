# BasicDriverFeatureStep

`BasicDriverFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`BasicDriverFeatureStep` takes the following to be created:

* <span id="conf"> [KubernetesDriverConf](KubernetesDriverConf.md)

`BasicDriverFeatureStep` is created when:

* `KubernetesDriverBuilder` is requested to [buildFromFeatures](KubernetesDriverBuilder.md#buildFromFeatures)

## <span id="getAdditionalPodSystemProperties"> Additional System Properties

```scala
getAdditionalPodSystemProperties(): Map[String, String]
```

`getAdditionalPodSystemProperties` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#getAdditionalPodSystemProperties) abstraction.

`getAdditionalPodSystemProperties` sets the following additional properties:

Name     | Value
---------|---------
 [spark.kubernetes.submitInDriver](configuration-properties.md#spark.kubernetes.submitInDriver) | `true`
 [spark.kubernetes.driver.pod.name](configuration-properties.md#spark.kubernetes.driver.pod.name) | [driverPodName](#driverPodName)
 [spark.kubernetes.memoryOverheadFactor](configuration-properties.md#spark.kubernetes.memoryOverheadFactor) | [overheadFactor](#overheadFactor)
 `spark.app.id` | [appId](KubernetesDriverConf.md#appId) (of the [KubernetesDriverConf](#conf))

In the end, `getAdditionalPodSystemProperties` [uploads local files](KubernetesUtils.md#uploadAndTransformFileUris) (in `spark.jars` and `spark.files` configuration properties) to a Hadoop-compatible file system and adds their target Hadoop paths to the additional properties.

!!! note "spark.kubernetes.file.upload.path Configuration Property"
    The Hadoop DFS path to upload local files to is defined using [spark.kubernetes.file.upload.path](configuration-properties.md#spark.kubernetes.file.upload.path) configuration property.

## <span id="configurePod"> Configuring Pod for Driver

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod`...FIXME

## <span id="driverContainerImage"> Driver Container Image Name

`BasicDriverFeatureStep` uses [spark.kubernetes.driver.container.image](configuration-properties.md#spark.kubernetes.driver.container.image) for the name of the container image for drivers.

The name must be defined or `BasicDriverFeatureStep` throws an `SparkException`:

```text
Must specify the driver container image
```

`driverContainerImage` is used when requested for [configurePod](#configurePod).
