# BasicDriverFeatureStep

`BasicDriverFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`BasicDriverFeatureStep` takes the following to be created:

* <span id="conf"> [KubernetesDriverConf](KubernetesDriverConf.md)

`BasicDriverFeatureStep` is created when:

* `KubernetesDriverBuilder` is requested to [build a driver pod spec](KubernetesDriverBuilder.md#buildFromFeatures)

## <span id="driverPodName"> Name of Driver Pod

`BasicDriverFeatureStep` uses the [spark.kubernetes.driver.pod.name](configuration-properties.md#spark.kubernetes.driver.pod.name) configuration property (if defined) or the [resourceNamePrefix](KubernetesDriverConf.md#resourceNamePrefix) (of the given [KubernetesDriverConf](#conf)) as the name of the driver pod:

```text
[resourceNamePrefix]-driver
```

## <span id="driverMemoryWithOverheadMiB"> Driver Memory

`BasicDriverFeatureStep` calculates the **driver memory** for the driver pod based on the [Base Driver Memory](#driverMemoryMiB) with the extra [Non-Heap Memory Overhead](#memoryOverheadMiB).

Driver memory is the quantity (in `Mi`s) of the driver memory resource (the request and limit).

??? tip "Demo: Running Spark Examples on minikube"
    Use `kubectl get po [driverPod]` to review the memory resource spec. Learn more in [Demo: Running Spark Examples on minikube](demo/running-spark-examples-on-minikube.md#container-resources).

### <span id="driverMemoryMiB"> Base Driver Memory

`BasicDriverFeatureStep` uses `spark.driver.memory` configuration property (default: `1g`) for the **base driver memory**.

### <span id="overheadFactor"> Memory Overhead Factor

`BasicDriverFeatureStep` uses different **memory overhead factors** based on the [MainAppResource](KubernetesDriverConf.md#mainAppResource) (of the given [KubernetesDriverConf](#conf)):

* For JVM applications (Java, Scala, etc.), it is [spark.kubernetes.memoryOverheadFactor](configuration-properties.md#spark.kubernetes.memoryOverheadFactor) configuration property

* For non-JVM applications (Python and R applications), it is [spark.kubernetes.memoryOverheadFactor](configuration-properties.md#spark.kubernetes.memoryOverheadFactor) configuration property (if defined) or `0.4`

`overheadFactor` is used for the following:

* [Additional System Properties](#getAdditionalPodSystemProperties)
* [Memory Overhead](#memoryOverheadMiB)

### <span id="memoryOverheadMiB"> Non-Heap Memory Overhead

`BasicDriverFeatureStep` defines the **memory overhead** for a non-heap memory to be allocated per driver in cluster mode (in `MiB`s) based on the following:

* `spark.driver.memoryOverhead` configuration property (if defined)
* Maximum of the [Memory Overhead Factor](#overheadFactor) multiplied by the [Base Driver Memory](#driverMemoryMiB) and `384`

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

## <span id="driverContainerImage"> Driver Container Image Name

`BasicDriverFeatureStep` uses [spark.kubernetes.driver.container.image](configuration-properties.md#spark.kubernetes.driver.container.image) for the name of the container image for drivers.

The name must be defined or `BasicDriverFeatureStep` throws an `SparkException`:

```text
Must specify the driver container image
```

`driverContainerImage` is used when requested for [configurePod](#configurePod).

## <span id="configurePod"> Configuring Driver Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod`...FIXME
