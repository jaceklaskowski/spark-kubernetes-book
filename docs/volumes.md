# Storage Volumes

**Storage Volumes** ([Kubernetes]({{ k8s.doc }}/concepts/storage/volumes/)) are configured using [MountVolumesFeatureStep](MountVolumesFeatureStep.md) and [KubernetesVolumeSpec](KubernetesVolumeSpec.md).

## Types of Volumes

Spark on Kubernetes supports the following types of volumes:

* `emptyDir`
* `hostPath`
* `nfs`
* `persistentVolumeClaim`

## <span id="KUBERNETES_DRIVER_VOLUMES_PREFIX"><span id="KUBERNETES_EXECUTOR_VOLUMES_PREFIX"> Volume Configuration Prefixes

Spark on Kubernetes uses the prefixes for volume-related configuration properties for driver and executor pods:

* `spark.kubernetes.driver.volumes.`
* `spark.kubernetes.executor.volumes.`

## Claim Name Placeholders

[MountVolumesFeatureStep](MountVolumesFeatureStep.md#claim-name-placeholders) defines **OnDemand** and **SPARK_EXECUTOR_ID** as placeholders for runtime-replaceable parts of the claim name of a [KubernetesPVCVolumeConf](KubernetesVolumeSpec.md#KubernetesPVCVolumeConf).

These placeholders allow for templating claim names to include parts to be replaced at deployment.

Follow [Demo: PersistentVolumeClaims](demo/persistentvolumeclaims.md) to see the feature in action.

### <span id="OnDemand"> OnDemand

`MountVolumesFeatureStep` replaces all `OnDemand`s with the following:

Pod          | Replacement
-------------|---------
 driver      | `[resourceNamePrefix]-exec-[executorId]-pvc-[volumeIndex]`
 executors   | `[resourceNamePrefix]-driver-pvc-[volumeIndex]`

### <span id="SPARK_EXECUTOR_ID"> SPARK_EXECUTOR_ID

`MountVolumesFeatureStep` replaces all `SPARK_EXECUTOR_ID`s with executor IDs.
