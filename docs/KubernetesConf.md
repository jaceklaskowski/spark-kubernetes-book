# KubernetesConf

`KubernetesConf` is an [abstraction](#contract) of [Kubernetes configuration metadata](#implementations) to build Spark pods (for the [driver](KubernetesDriverConf.md) and [executors](KubernetesExecutorConf.md)).

## Contract

### <span id="annotations"> annotations

```scala
annotations: Map[String, String]
```

Used when:

* `BasicDriverFeatureStep` is requested to [configurePod](BasicDriverFeatureStep.md#configurePod)
* `BasicExecutorFeatureStep` is requested to [configurePod](BasicExecutorFeatureStep.md#configurePod)

### <span id="environment"> environment

```scala
environment: Map[String, String]
```

Used when:

* `BasicDriverFeatureStep` is requested to [configurePod](BasicDriverFeatureStep.md#configurePod)
* `BasicExecutorFeatureStep` is requested to [configurePod](BasicExecutorFeatureStep.md#configurePod)

### <span id="labels"> labels

```scala
labels: Map[String, String]
```

Used when:

* `BasicDriverFeatureStep` is requested to [configurePod](BasicDriverFeatureStep.md#configurePod)
* `BasicExecutorFeatureStep` is requested to [configurePod](BasicExecutorFeatureStep.md#configurePod)
* `DriverServiceFeatureStep` is requested to [getAdditionalKubernetesResources](DriverServiceFeatureStep.md#getAdditionalKubernetesResources)

### <span id="resourceNamePrefix"> resourceNamePrefix

```scala
resourceNamePrefix: String
```

Prefix of resource names

### <span id="secretEnvNamesToKeyRefs"> secretEnvNamesToKeyRefs

```scala
secretEnvNamesToKeyRefs: Map[String, String]
```

Used when:

* `EnvSecretsFeatureStep` is requested to [configurePod](EnvSecretsFeatureStep.md#configurePod)

### <span id="secretNamesToMountPaths"> secretNamesToMountPaths

```scala
secretNamesToMountPaths: Map[String, String]
```

Used when:

* `MountSecretsFeatureStep` is requested to [configurePod](MountSecretsFeatureStep.md#configurePod)

### <span id="volumes"> Volumes

```scala
volumes: Seq[KubernetesVolumeSpec]
```

[KubernetesVolumeSpec](KubernetesVolumeSpec.md)s

Used when:

* `MountVolumesFeatureStep` is requested to [configure a pod](MountVolumesFeatureStep.md#configurePod)

## Implementations

* [KubernetesDriverConf](KubernetesDriverConf.md)
* [KubernetesExecutorConf](KubernetesExecutorConf.md)

## Creating Instance

`KubernetesConf` takes the following to be created:

* <span id="sparkConf"> `SparkConf`

??? note "Abstract Class"
    `KubernetesConf` is an abstract class and cannot be created directly. It is created indirectly for the [concrete KubernetesConfs](#implementations).

## <span id="namespace"> Namespace

```scala
namespace: String
```

`namespace` is the value of [spark.kubernetes.namespace](configuration-properties.md#spark.kubernetes.namespace) configuration property.

`namespace` is used when:

* `DriverServiceFeatureStep` is requested to [getAdditionalPodSystemProperties](DriverServiceFeatureStep.md#getAdditionalPodSystemProperties)
* `Client` is requested to [run](Client.md#run)
* `KubernetesClientApplication` is requested to [start](KubernetesClientApplication.md#start)

## <span id="imagePullPolicy"> imagePullPolicy

```scala
imagePullPolicy: String
```

`imagePullPolicy` is the value of [spark.kubernetes.container.image.pullPolicy](configuration-properties.md#spark.kubernetes.container.image.pullPolicy) configuration property.

`imagePullPolicy` is used when:

* `BasicDriverFeatureStep` is requested to [configure a pod](BasicDriverFeatureStep.md#configurePod)
* `BasicExecutorFeatureStep` is requested to [configure a pod](BasicExecutorFeatureStep.md#configurePod)

## <span id="createDriverConf"> Creating KubernetesDriverConf

```scala
createDriverConf(
  sparkConf: SparkConf,
  appId: String,
  mainAppResource: MainAppResource,
  mainClass: String,
  appArgs: Array[String]): KubernetesDriverConf
```

!!! note
    The goal of `createDriverConf` is to validate executor volumes before creating a [KubernetesDriverConf](KubernetesDriverConf.md).

`createDriverConf` [parse volumes](KubernetesVolumeUtils.md#parseVolumesWithPrefix) for executors (with **spark.kubernetes.executor.volumes** prefix).

!!! note
    `createDriverConf` parses executor volumes in order to verify configuration before the driver pod is created.

In the end, `createDriverConf` creates a [KubernetesDriverConf](KubernetesDriverConf.md).

`createDriverConf` is used when:

* `KubernetesClientApplication` is requested to [start](KubernetesClientApplication.md#start)

## <span id="createExecutorConf"> Creating KubernetesExecutorConf

```scala
createExecutorConf(
  sparkConf: SparkConf,
  executorId: String,
  appId: String,
  driverPod: Option[Pod]): KubernetesExecutorConf
```

`createExecutorConf` (_does nothing more but_) creates a [KubernetesExecutorConf](KubernetesExecutorConf.md) for the given input arguments.

`createExecutorConf` is used when:

* `ExecutorPodsAllocator` is requested to [onNewSnapshots](ExecutorPodsAllocator.md#onNewSnapshots) (and requests missing executors from Kubernetes)

## <span id="getResourceNamePrefix"> AppName-Based Unique Resource Name Prefix

```scala
getResourceNamePrefix(
  appName: String): String
```

`getResourceNamePrefix`...FIXME

`getResourceNamePrefix` is used when:

* `KubernetesDriverConf` is requested for the [resourceNamePrefix](KubernetesDriverConf.md#resourceNamePrefix)
* `KubernetesExecutorConf` is requested for the [resourceNamePrefix](KubernetesExecutorConf.md#resourceNamePrefix)
* `KubernetesClusterManager` is requested to [createSchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)

## <span id="buildKubernetesResourceName"> Building Kubernetes Resource Name

```scala
buildKubernetesResourceName(
  vendorDomain: String,
  resourceName: String): String
```

`buildKubernetesResourceName` creates the following text (resource name):

```text
[vendorDomain]/[resourceName]
```

`buildKubernetesResourceName` is used when:

* `KubernetesUtils` is requested to [buildResourcesQuantities](KubernetesUtils.md#buildResourcesQuantities)
* `BasicExecutorFeatureStep` is requested to [buildExecutorResourcesQuantities](BasicExecutorFeatureStep.md#buildExecutorResourcesQuantities)
