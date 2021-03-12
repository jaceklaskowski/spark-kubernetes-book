# KubernetesVolumeUtils

`KubernetesVolumeUtils` utility is used to [parseVolumesWithPrefix](#parseVolumesWithPrefix).

## <span id="parseVolumesWithPrefix"> parseVolumesWithPrefix

```scala
parseVolumesWithPrefix(
  sparkConf: SparkConf,
  prefix: String): Seq[KubernetesVolumeSpec]
```

`parseVolumesWithPrefix` creates [KubernetesVolumeSpec](KubernetesVolumeSpec.md)s based on the configuration properties (in the given `SparkConf`) with the `prefix`.

`parseVolumesWithPrefix` requests the `SparkConf` to get all properties with the given `prefix`.

`parseVolumesWithPrefix` [extracts volume types and names](#getVolumeTypesAndNames) (from the keys of the properties) and for every pair creates a [KubernetesVolumeSpec](KubernetesVolumeSpec.md) with the following:

* `volumeName`
* `mountPath` based on `[volumeType].[volumeName].mount.path` key (in the properties)
* `mountSubPath` based on `[volumeType].[volumeName].mount.subPath` key (in the properties) if available or defaults to an empty path
* `mountReadOnly` based on `[volumeType].[volumeName].mount.readOnly` key (in the properties) if available or `false`
* `volumeConf` with a [KubernetesVolumeSpecificConf](#parseVolumeSpecificConf) based on the properties and the `volumeType` and `volumeName` of the volume

`parseVolumesWithPrefix` is used when:

* `KubernetesDriverConf` is requested for [volumes](KubernetesDriverConf.md#volumes)
* `KubernetesExecutorConf` is requested for [volumes](KubernetesExecutorConf.md#volumes)
* `KubernetesConf` utility is used to [create a KubernetesDriverConf](KubernetesConf.md#createDriverConf)

## <span id="getVolumeTypesAndNames"> getVolumeTypesAndNames

```scala
getVolumeTypesAndNames(
  properties: Map[String, String]): Set[(String, String)]
```

`getVolumeTypesAndNames` extracts volume types and names.

`getVolumeTypesAndNames` splits the keys (in the given `properties` key-value collection) by `.` to volume type and name pairs.

## <span id="parseVolumeSpecificConf"> parseVolumeSpecificConf

```scala
parseVolumeSpecificConf(
  options: Map[String, String],
  volumeType: String,
  volumeName: String): KubernetesVolumeSpecificConf
```

`parseVolumeSpecificConf` extracts volume configuration given the volume type and name.

`parseVolumeSpecificConf` creates a [KubernetesVolumeSpecificConf](KubernetesVolumeSpec.md#KubernetesVolumeSpecificConf) based on the given `volumeType`.

`parseVolumeSpecificConf` throws an `IllegalArgumentException` for unsupported `volumeType`:

```text
Kubernetes Volume type `[volumeType]` is not supported
```
