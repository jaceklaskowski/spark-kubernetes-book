---
hide:
  - navigation
---

# Demo: PersistentVolumeClaims

This demo shows how to use [PersistentVolumeClaims](../volumes.md).

From [Persistent Volumes]({{ minikube.docs }}/handbook/persistent_volumes/):

> minikube supports PersistentVolumes of type hostPath out of the box.
> These PersistentVolumes are mapped to a directory inside the running minikube instance

The demo uses [OnDemand](../volumes.md#OnDemand) claim name placeholder to create different PersistentVolume claim names for the executors at deployment.

From [Binding]({{ minikube.docs }}/handbook/persistent_volumes/#binding):

> Once bound, PersistentVolumeClaim binds are exclusive, regardless of how they were bound.
> A PVC to PV binding is a one-to-one mapping, using a ClaimRef which is a bi-directional binding between the PersistentVolume and the PersistentVolumeClaim.
>
> Claims will remain unbound indefinitely if a matching volume does not exist.

Given the demo uses 2 executors you could simply create 2 persistent volumes and be done. More executors would beg for some automation and Kubernetes supports this use case with [Dynamic Volume Provisioning]({{ minikube.docs }}/concepts/storage/dynamic-provisioning/):

> Dynamic volume provisioning allows storage volumes to be created on-demand.

In this demo you simply create two PVs to get going.

## Before you begin

It is assumed that you have finished the following:

- [Demo: Running Spark Application on minikube](running-spark-application-on-minikube.md)

## Start Cluster

```text
minikube start
```

Switch the Kubernetes namespace to ours.

```text
kubens spark-demo
```

## Environment Variables

```text
export K8S_SERVER=$(k config view --output=jsonpath='{.clusters[].cluster.server}')

export POD_NAME=meetup-spark-app
export IMAGE_NAME=$POD_NAME:0.1.0

export VOLUME_NAME=my-pv
export MOUNT_PATH=/mnt/data
export PVC_STORAGE_CLASS=manual
export PVC_SIZE_LIMIT=1Gi
```

## Create PersistentVolumes

The following commands are a copy of [Configure a Pod to Use a PersistentVolume for Storage]({{ k8s.doc }}/tasks/configure-pod-container/configure-persistent-volume-storage/#create-an-index-html-file-on-your-node) (with some minor changes).

```text
minikube ssh
```

```text
sudo mkdir /data/pv0001
```

```text
sudo sh -c "echo 'Hello from Kubernetes storage' > /data/pv0001/hello-message"
```

Create a PersistentVolume as described in [Configure a Pod to Use a PersistentVolume for Storage]({{ k8s.doc }}//tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume).

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 2Gi
  hostPath:
    path: /data/pv0001/
  storageClassName: manual
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0002
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 2Gi
  hostPath:
    path: /data/pv0001/
  storageClassName: manual
```

```text
k apply -f k8s/pvs.yaml
```

```text
k get pv
```

```text
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv0001   2Gi        RWO            Retain           Available           manual                  5s
pv0002   2Gi        RWO            Retain           Available           manual                  5s
```

```text
k describe pv pv0001
```

```text
Name:            pv0001
Labels:          <none>
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    manual
Status:          Available
Claim:
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        2Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /data/pv0001/
    HostPathType:
Events:            <none>
```

## Claim Persistent Volume

You're going to use `spark-shell` Spark application and executors only are provisioned on Kubernetes.

```text
cd $SPARK_HOME
```

Spark on Kubernetes sets up Kubernetes' [persistentVolumeClaim]({{ k8s.doc }}/concepts/storage/volumes/#persistentvolumeclaim) using `spark.kubernetes.executor.volumes`-prefixed configuration properties for executor pods.

Spark on Kubernetes uses 2 executors by default (`--num-executors 2`) and that is why the demo uses `OnDemand` claim name to generate different PV claim names at deployment.

### Watch Persistent Volume Claims

In a separate terminal use the following command to watch persistent volume claims as they are created.

```text
k get pvc -w
```

The output will be similar to the following:

```text
NAME                                        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
spark-shell-de6ed2783a61d962-exec-1-pvc-0   Pending                                      manual         0s
spark-shell-de6ed2783a61d962-exec-1-pvc-0   Pending   pv0001   0                         manual         0s
spark-shell-de6ed2783a61d962-exec-1-pvc-0   Bound     pv0001   2Gi        RWO            manual         0s
spark-shell-de6ed2783a61d962-exec-2-pvc-0   Pending                                      manual         0s
spark-shell-de6ed2783a61d962-exec-2-pvc-0   Pending   pv0002   0                         manual         0s
spark-shell-de6ed2783a61d962-exec-2-pvc-0   Bound     pv0002   2Gi        RWO            manual         0s
```

### Start Spark Shell

``` shell
./bin/spark-shell \
  --master k8s://$K8S_SERVER \
  --conf spark.kubernetes.executor.volumes.persistentVolumeClaim.$VOLUME_NAME.mount.path=$MOUNT_PATH \
  --conf spark.kubernetes.executor.volumes.persistentVolumeClaim.$VOLUME_NAME.options.claimName=OnDemand \
  --conf spark.kubernetes.executor.volumes.persistentVolumeClaim.$VOLUME_NAME.options.storageClass=$PVC_STORAGE_CLASS \
  --conf spark.kubernetes.executor.volumes.persistentVolumeClaim.$VOLUME_NAME.options.sizeLimit=$PVC_SIZE_LIMIT \
  --conf spark.kubernetes.container.image=$IMAGE_NAME \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose
```

!!! note
    You have to recreate the persistent volumes ([Clean Up](#clean-up) and [Create PersistentVolumes](#create-persistentvolumes)) before running `spark-shell` again (per [Retain]({{ k8s.doc }}/concepts/storage/persistent-volumes/#retain) reclaim policy).

## Review Kubernetes Resources

### persistentVolumes

```text
k describe pv $(k get pv -o=jsonpath='{.items[0].metadata.name}')
```

```text
Name:            pv0001
Labels:          <none>
Annotations:     pv.kubernetes.io/bound-by-controller: yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    manual
Status:          Bound
Claim:           spark-demo/spark-shell-de6ed2783a61d962-exec-1-pvc-0
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        2Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /data/pv0001/
    HostPathType:
Events:            <none>
```

### persistentVolumeClaims

```text
k get pvc
```

```text
NAME                                        STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
spark-shell-de6ed2783a61d962-exec-1-pvc-0   Bound    pv0001   2Gi        RWO            manual         2m19s
spark-shell-de6ed2783a61d962-exec-2-pvc-0   Bound    pv0002   2Gi        RWO            manual         2m19s
```

```text
k describe pvc $(k get pvc -o=jsonpath='{.items[0].metadata.name}')
```

```text
Name:          spark-shell-de6ed2783a61d962-exec-1-pvc-0
Namespace:     spark-demo
StorageClass:  manual
Status:        Bound
Volume:        pv0001
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      2Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       spark-shell-de6ed2783a61d962-exec-1
Events:        <none>
```

## Access PersistentVolume

### Command Line

```text
k exec -ti $(k get po -o name) -- cat $MOUNT_PATH/hello-message
```

```text
Hello from Kubernetes storage
```

### Spark Shell

!!! note
    Wish I knew how to show that the only executor has got access to the mounted volume, but nothing comes to my mind. If you happen to know how to demo it, please contact me at jacek@japila.pl. Thank you! ❤️

## Clean Up

```text
k delete po --all
k delete pvc --all
k delete pv --all
```

Clean up the cluster as described in [Demo: spark-shell on minikube](spark-shell-on-minikube.md#clean-up).

_That's it. Congratulations!_
