# DriverCommandFeatureStep

`DriverCommandFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`DriverCommandFeatureStep` takes the following to be created:

* <span id="conf"> [KubernetesDriverConf](KubernetesDriverConf.md)

`DriverCommandFeatureStep` is created when:

* `KubernetesDriverBuilder` is requested to [build a KubernetesDriverSpec from features](KubernetesDriverBuilder.md#buildFromFeatures)

## <span id="configurePod"> Configuring Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod` branches off based on the [MainAppResource](KubernetesDriverConf.md#mainAppResource) (of the [KubernetesDriverConf](#conf)):

* For `JavaMainAppResource`, `configurePod` [configures a pod for Java](#configureForJava) with the primary resource (if defined) or uses `spark-internal` special value

* For `PythonMainAppResource`, `configurePod` [configures a pod for Python](#configureForPython) with the primary resource

* For `RMainAppResource`, `configurePod` [configures a pod for R](#configureForR) with the primary resource

### <span id="configureForJava"> Configuring Pod for Java Application

```scala
configureForJava(
  pod: SparkPod,
  res: String): SparkPod
```

`configureForJava` builds the [base driver container](#baseDriverContainer) for the given `SparkPod` and the primary resource.

In the end, `configureForJava` creates another `SparkPod` (for the pod of the given `SparkPod`) and the driver container.

### <span id="configureForPython"> Configuring Pod for Python Application

```scala
configureForPython(
  pod: SparkPod,
  res: String): SparkPod
```

`configureForPython`...FIXME

### <span id="configureForR"> Configuring Pod for R Application

```scala
configureForR(
  pod: SparkPod,
  res: String): SparkPod
```

`configureForR`...FIXME

### <span id="baseDriverContainer"> Base Driver ContainerBuilder

```scala
baseDriverContainer(
  pod: SparkPod,
  resource: String): ContainerBuilder
```

`baseDriverContainer` [renames](KubernetesUtils.md#renameMainAppResource) the given primary `resource` if the [MainAppResource](KubernetesDriverConf.md#mainAppResource) is a `JavaMainAppResource`. Otherwise, `baseDriverContainer` leaves the primary resource as-is.

`baseDriverContainer` creates a `ContainerBuilder` (for the pod of the given `SparkPod`) and adds the following arguments (in that order):

1. `driver`
1. `--properties-file` with `/opt/spark/conf/spark.properties`
1. `--class` with the [mainClass](KubernetesDriverConf.md#mainClass) of the [KubernetesDriverConf](#conf)
1. the primary resource (possibly [renamed](KubernetesUtils.md#renameMainAppResource) if a `MainAppResource`)
1. [appArgs](KubernetesDriverConf.md#appArgs) of the [KubernetesDriverConf](#conf)

!!! note
    The arguments are then used by the default `entrypoint.sh` of the official Docker image of Apache Spark (in `resource-managers/kubernetes/docker/src/main/dockerfiles/spark/`).

    Use the following `kubectl` command to see the arguments:

    ```text
    kubectl get po [driverPod] -o=jsonpath='{.spec.containers[0].args}'
    ```
