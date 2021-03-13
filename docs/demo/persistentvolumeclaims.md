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

## Environment Variables

```text
export K8S_SERVER=$(k config view --output=jsonpath='{.clusters[].cluster.server}')
export POD_NAME=meetup-spark-app
export IMAGE_NAME=$POD_NAME:0.1.0

export VOLUME_NAME=my-pvc
export MOUNT_PATH=/my-pvc
export PVC_CLAIM_NAME=my-claim-name
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

```text
k apply -f https://k8s.io/examples/pods/storage/pv-volume.yaml
```

```text
k get pv
```

```text
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
task-pv-volume   10Gi       RWO            Retain           Available           manual                  21s
```

## Using persistentVolumeClaim

Let's use Kubernetes' [persistentVolumeClaim]({{ k8s.doc }}/concepts/storage/volumes/#persistentvolumeclaim) that requires `spark.kubernetes.*.volumes`-prefixed configuration properties for the driver and executor pods:

```text
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.$VOLUME_NAME.options.claimName=$VOLUME_CLAIM_NAME
```

```text
cd $SPARK_HOME
```

```text
./bin/spark-shell \
  --master k8s://$K8S_SERVER \
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

```text
k get pvc
```

## Clean Up

```text
k delete pvc --all
k delete pv --all
k delete po --all
```

Clean up the cluster as described in [Demo: spark-shell on minikube](spark-shell-on-minikube.md#clean-up).

_That's it. Congratulations!_
