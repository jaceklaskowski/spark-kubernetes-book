# KubernetesVolumeSpec

`KubernetesVolumeSpec` is a Kubernetes volume specification:

* <span id="volumeName"> Volume Name
* <span id="mountPath"> Mount Path
* <span id="mountSubPath"> Mount SubPath (default: empty)
* <span id="mountReadOnly"> `readOnly` flag (default: `false`)
* <span id="volumeConf"> [KubernetesVolumeSpecificConf](#KubernetesVolumeSpecificConf)

`KubernetesVolumeSpec` is part of [KubernetesConf](KubernetesConf.md#volumes) abstraction.

`KubernetesVolumeSpec` is created using [KubernetesVolumeUtils](KubernetesVolumeUtils.md#parseVolumesWithPrefix) utility.

## <span id="KubernetesVolumeSpecificConf"> KubernetesVolumeSpecificConf

`KubernetesVolumeSpecificConf` complements `KubernetesVolumeSpec` with a [volume type-specific configuration](#volumeConf).

`KubernetesVolumeSpecificConf` is created using [KubernetesVolumeUtils](KubernetesVolumeUtils.md#parseVolumeSpecificConf) utility.

??? note "Sealed Trait"
    `KubernetesVolumeSpecificConf` is a Scala **sealed trait** which means that all of the implementations are in the same compilation unit (a single file).

## <span id="KubernetesEmptyDirVolumeConf"> KubernetesEmptyDirVolumeConf

* Optional `medium`
* Optional `sizeLimit`

## <span id="KubernetesHostPathVolumeConf"> KubernetesHostPathVolumeConf

* `hostPath` (based on `hostPath.[volumeName].options.path` configuration property)

`KubernetesHostPathVolumeConf` is used when:

* `MountVolumesFeatureStep` is requested to [constructVolumes](MountVolumesFeatureStep.md#constructVolumes) (that creates a hostPath volume for the given `hostPath` volume source path and an empty type)

## <span id="KubernetesNFSVolumeConf"> KubernetesNFSVolumeConf

* `path`
* `server`

## <span id="KubernetesPVCVolumeConf"> KubernetesPVCVolumeConf

`KubernetesPVCVolumeConf` is a [KubernetesVolumeSpecificConf](#KubernetesVolumeSpecificConf) with the following:

* `claimName`
* Optional `storageClass` (default: undefined)
* Optional `size` (default: undefined)

`KubernetesPVCVolumeConf` is created (using [KubernetesVolumeUtils](KubernetesVolumeUtils.md#parseVolumeSpecificConf) utility) based on configuration properties with `persistentVolumeClaim` volume type prefix.

Attribute       | Configuration Property
----------------|-----------------------
 `claimName`    | `persistentVolumeClaim.[volumeName].options.claimName`
 `storageClass` | `persistentVolumeClaim.[volumeName].options.storageClass`
 `size`         | `persistentVolumeClaim.[volumeName].options.sizeLimit`

`KubernetesPVCVolumeConf` is used when:

* `MountVolumesFeatureStep` is requested to [constructVolumes](MountVolumesFeatureStep.md#constructVolumes) (that creates a [PersistentVolumeClaim]({{ k8s.doc }}/concepts/storage/persistent-volumes/) for the given `claimName`)
