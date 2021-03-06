# MountVolumesFeatureStep

`MountVolumesFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`MountVolumesFeatureStep` takes the following to be created:

* <span id="conf"> [KubernetesConf](KubernetesConf.md)

`MountVolumesFeatureStep` is created when:

* `KubernetesDriverBuilder` is requested to [build a driver pod spec](KubernetesDriverBuilder.md#buildFromFeatures)
* `KubernetesExecutorBuilder` is requested for a [pod spec for executors](KubernetesExecutorBuilder.md#buildFromFeatures)

## <span id="configurePod"> Configuring Driver Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod` [constructs the volumes and volume mounts](#constructVolumes) from the [volumes](KubernetesConf.md#volumes) (of the [KubernetesConf](#conf)) and creates a new `SparkPod`:

* Adds the volumes to the pod specification

* Adds the volume mounts to the container specification

### <span id="constructVolumes"> constructVolumes

```scala
constructVolumes(
  volumeSpecs: Iterable[KubernetesVolumeSpec]): Iterable[(VolumeMount, Volume)]
```

`constructVolumes` creates Kubernetes `VolumeMount`s and `Volume`s for the given [KubernetesVolumeSpec](KubernetesVolumeSpec.md)s.

`VolumeMount`s are built based on the following (4 of 5 properties):

* [mountPath](KubernetesVolumeSpec.md#mountPath)
* [mountReadOnly](KubernetesVolumeSpec.md#mountReadOnly)
* [mountSubPath](KubernetesVolumeSpec.md#mountSubPath)
* [volumeName](KubernetesVolumeSpec.md#volumeName)

`Volume`s are build based on the [type of the volume](KubernetesVolumeSpec.md#volumeConf) (the remaining 5th property).

In the end, `Volume`s and `VolumeMount`s are wired together using `volumeName`.

## <span id="OnDemand"><span id="PVC_ON_DEMAND"><span id="SPARK_EXECUTOR_ID"> Claim Name Placeholders

`MountVolumesFeatureStep` defines **OnDemand** and **SPARK_EXECUTOR_ID** as placeholders for runtime-replaceable parts of the claim name of a [KubernetesPVCVolumeConf](KubernetesVolumeSpec.md#KubernetesPVCVolumeConf).

These placeholders allow for templating claim names to include parts to be replaced at deployment.

When [constructVolumes](#constructVolumes) `MountVolumesFeatureStep` replaces all `OnDemand`s with the following (using [resourceNamePrefix](KubernetesConf.md#resourceNamePrefix) and [executorId](KubernetesExecutorConf.md#executorId) of the [KubernetesConf](#conf)):

Pod          | Replacement
-------------|---------
 driver      | `[resourceNamePrefix]-exec-[executorId]-pvc-[volumeIndex]`
 executors   | `[resourceNamePrefix]-driver-pvc-[volumeIndex]`

When [constructVolumes](#constructVolumes) `MountVolumesFeatureStep` replaces all `SPARK_EXECUTOR_ID`s with [executorId](KubernetesExecutorConf.md#executorId) of the [KubernetesConf](#conf).
