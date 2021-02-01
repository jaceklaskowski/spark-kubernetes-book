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

`constructVolumes` creates Kubernetes `VolumeMount`s and `Volume`s based on the given `KubernetesVolumeSpec` specs.

`VolumeMount`s are built based on the following:

* `mountPath`
* `mountReadOnly`
* `mountSubPath`
* `volumeName`

`Volume`s are build based on the type of the volume:

* `HostPath`
* `PersistentVolumeClaim`
* `EmptyDir`

In the end, `Volume`s and `VolumeMount`s are wired together using `volumeName`.
