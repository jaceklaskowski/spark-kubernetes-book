# Storage Volumes

**Storage Volumes** ([Kubernetes]({{ k8s.doc }}/concepts/storage/volumes/)) are configured using [MountVolumesFeatureStep](MountVolumesFeatureStep.md) using [KubernetesVolumeSpec](KubernetesVolumeSpec.md).

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
