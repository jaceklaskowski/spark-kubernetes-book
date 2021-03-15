---
hide:
  - navigation
---

# Demo: PersistentVolumeClaims

This demo shows how to use [PersistentVolumeClaims](../volumes.md).

From [Persistent Volumes]({{ minikube.docs }}/handbook/persistent_volumes/):

> minikube supports PersistentVolumes of type hostPath out of the box.
> These PersistentVolumes are mapped to a directory inside the running minikube instance

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
export PVC_CLAIM_NAME=my-pv-claim
export PVC_STORAGE_CLASS=manual
export PVC_SIZE_LIMIT=1Gi
```

## Create PersistentVolume

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
    storage: 5Gi
  hostPath:
    path: /data/pv0001/
  storageClassName: manual
```

```text
k apply -f k8s/pv-volume.yaml
```

```text
k get pv
```

```text
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv0001   5Gi        RWO            Retain           Available           manual                  4s
```

## Claim Persistent Volume

You're going to use `spark-shell` Spark application and executors only are provisioned on Kubernetes.

Spark on Kubernetes sets up Kubernetes' [persistentVolumeClaim]({{ k8s.doc }}/concepts/storage/volumes/#persistentvolumeclaim) using `spark.kubernetes.*.volumes`-prefixed configuration properties (for the driver and executor pods, but you need executors only).

```text
cd $SPARK_HOME
```

Please note the demo uses 1 executor or else you face _"persistentvolumeclaims "my-pvc" already exists"_ failure in the logs (_FIXME_).

``` shell
./bin/spark-shell \
  --master k8s://$K8S_SERVER \
  --num-executors 1 \
  --conf spark.kubernetes.executor.volumes.persistentVolumeClaim.$VOLUME_NAME.mount.path=$MOUNT_PATH \
  --conf spark.kubernetes.executor.volumes.persistentVolumeClaim.$VOLUME_NAME.options.claimName=$PVC_CLAIM_NAME \
  --conf spark.kubernetes.executor.volumes.persistentVolumeClaim.$VOLUME_NAME.options.storageClass=$PVC_STORAGE_CLASS \
  --conf spark.kubernetes.executor.volumes.persistentVolumeClaim.$VOLUME_NAME.options.sizeLimit=$PVC_SIZE_LIMIT \
  --conf spark.kubernetes.container.image=$IMAGE_NAME \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose
```

!!! important
    [Clean up](#clean-up) the persistent volume and the claim before running `spark-shell` again (per [Retain]({{ k8s.doc }}/concepts/storage/persistent-volumes/#retain) reclaim policy).

## Review Kubernetes Resources

### persistentVolumeClaim

```text
k describe pvc my-pv-claim
```

```text
Name:          my-pv-claim
Namespace:     spark-demo
StorageClass:  manual
Status:        Bound
Volume:        pv0001
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      5Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       spark-shell-f4ca607836d59a6d-exec-1
Events:        <none>
```

### persistentVolume

```text
k describe pv pv0001
```

```text
Name:            pv0001
Labels:          <none>
Annotations:     pv.kubernetes.io/bound-by-controller: yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    manual
Status:          Bound
Claim:           spark-demo/my-pv-claim
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        5Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /data/pv0001/
    HostPathType:
Events:            <none>
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
    Wish I knew how to show that the only executor has got access to the mounted volume, but nothing comes to my mind. If you happen to know how to demo it, please contact me at jacek@japila.pl. Thank you 🤙

## Clean Up

```text
k delete po --all
k delete pvc --all
k delete pv --all
```

Clean up the cluster as described in [Demo: spark-shell on minikube](spark-shell-on-minikube.md#clean-up).

_That's it. Congratulations!_
