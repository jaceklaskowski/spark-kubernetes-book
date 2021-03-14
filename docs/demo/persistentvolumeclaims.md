---
hide:
  - navigation
---

# Demo: PersistentVolumeClaims

This demo shows how to use [PersistentVolumeClaims](../volumes.md).

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

export VOLUME_NAME=my-pvc
export MOUNT_PATH=/my-pvc
export PVC_CLAIM_NAME=$VOLUME_NAME
export PVC_STORAGE_CLASS=manual
export PVC_SIZE_LIMIT=3Gi
```

## Create PersistentVolume

The following commands are copied from [Configure a Pod to Use a PersistentVolume for Storage]({{ k8s.doc }}/tasks/configure-pod-container/configure-persistent-volume-storage/#create-an-index-html-file-on-your-node).

```text
minikube ssh
```

```text
sudo mkdir /mnt/data
```

```text
sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
```

Create a PersistentVolume as described in [Configure a Pod to Use a PersistentVolume for Storage]({{ k8s.doc }}//tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume).

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

```text
k apply -f k8s/pv-volume.yaml
```

```text
k get pv
```

```text
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
task-pv-volume   10Gi       RWO            Retain           Available           manual                  21s
```

## Claim Persistent Volume

You're going to use `spark-shell` Spark application and executors only are provisioned on Kubernetes.

Spark on Kubernetes sets up Kubernetes' [persistentVolumeClaim]({{ k8s.doc }}/concepts/storage/volumes/#persistentvolumeclaim) using `spark.kubernetes.*.volumes`-prefixed configuration properties (for the driver and executor pods, but you need executors only).

```text
cd $SPARK_HOME
```

Please note the demo uses 1 executor to make demo slightly easier. It's optional and the default 2 executors should be fine too.

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

## Review Kubernetes Resources

### persistentVolumeClaim

```text
k get pvc
```

```text
NAME     STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc   Bound    task-pv-volume   10Gi       RWO            manual         42s
```

### persistentVolume

```text
k get pv
```

```text
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
task-pv-volume   10Gi       RWO            Retain           Bound    spark-demo/my-pvc   manual                  5m57s
```

## Access PersistentVolume

```text
k exec -ti $(k get po -o name) -- cat /my-pvc/index.html
```

## Clean Up

```text
k delete pvc --all
k delete pv --all
k delete po --all
```

Clean up the cluster as described in [Demo: spark-shell on minikube](spark-shell-on-minikube.md#clean-up).

_That's it. Congratulations!_
