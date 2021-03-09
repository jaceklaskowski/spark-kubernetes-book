# KubernetesVolumeSpec

`KubernetesVolumeSpec` is a Kubernetes volume specification:

* <span id="volumeName"> Volume Name
* <span id="mountPath"> Mount Path
* <span id="mountSubPath"> Mount SubPath
* <span id="mountReadOnly"> `mountReadOnly` flag
* <span id="volumeConf"> [KubernetesVolumeSpecificConf](#KubernetesVolumeSpecificConf)

## <span id="KubernetesVolumeSpecificConf"> KubernetesVolumeSpecificConf

`KubernetesVolumeSpecificConf` complements `KubernetesVolumeSpec` with a [volume type-specific configuration](#volumeConf).

`KubernetesVolumeSpecificConf` is created using [KubernetesVolumeUtils.parseVolumeSpecificConf](KubernetesVolumeUtils.md#parseVolumeSpecificConf) utility.

??? note "Sealed Trait"
    `KubernetesVolumeSpecificConf` is a Scala **sealed trait** which means that all of the implementations are in the same compilation unit (a single file).

### <span id="KubernetesEmptyDirVolumeConf"> KubernetesEmptyDirVolumeConf

* Optional `medium`
* Optional `sizeLimit`

### <span id="KubernetesHostPathVolumeConf"> KubernetesHostPathVolumeConf

* `hostPath` (based on `hostPath.[volumeName].options.path` configuration property)

`KubernetesHostPathVolumeConf` is used when:

* `MountVolumesFeatureStep` is requested to [constructVolumes](MountVolumesFeatureStep.md#constructVolumes) (that creates a hostPath volume for the given `hostPath` volume source path and an empty type)

### <span id="KubernetesNFSVolumeConf"> KubernetesNFSVolumeConf

* `path`
* `server`

### <span id="KubernetesPVCVolumeConf"> KubernetesPVCVolumeConf

* `claimName`
* Optional `storageClass` (default: undefined)
* Optional `size` (default: undefined)
