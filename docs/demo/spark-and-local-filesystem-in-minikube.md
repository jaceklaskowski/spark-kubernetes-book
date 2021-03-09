---
hide:
  - navigation
---

# Demo: Spark and Local Filesystem in minikube

The demo shows how to set up a Spark application on minikube to access files on a local filesystem.

!!! danger
    The demo stopped working for some reasons I cannot explain and sort out. The error message is the following:

    ```text
    Warning  Failed     13s   kubelet            Error: failed to start container "spark-kubernetes-driver": Error response from daemon: error while creating mount source path '/tmp/spark-k8s-demo/mount': mkdir /tmp/spark-k8s-demo: file exists
    ```

The demo uses `spark-submit --files` and [spark.kubernetes.file.upload.path](../configuration-properties.md#spark.kubernetes.file.upload.path) configuration property to upload a static file to a directory that is then mounted to Spark application pods.

**Volumes** in [Kubernetes]({{ k8s.doc }}/concepts/storage/volumes/) are directories which are accessible to the containers in a pod. In order to use a volume, you should specify the volumes to provide for the Pod in `.spec.volumes` and declare where to mount those volumes into containers in `.spec.containers[*].volumeMounts`.

## Before you begin

It is assumed that you have finished the following:

- [Demo: Running Spark Application on minikube](running-spark-application-on-minikube.md)

## Start Cluster

Unless already started, start minikube.

```text
minikube start
```

## Environment Variables

```text
export K8S_SERVER=$(k config view --output=jsonpath='{.clusters[].cluster.server}')
export POD_NAME=meetup-spark-app
export IMAGE_NAME=$POD_NAME:0.1.0

export SOURCE_DIR=/tmp/spark-k8s-demo
export VOLUME_TYPE=hostPath
export VOLUME_NAME=demo-host-mount
export MOUNT_PATH=$SOURCE_DIR/mount
```

## Mounting Filesystems

Let's kick things off by mounting a host directory (`/tmp/spark-k8s`) to minikube.

Quoting [Mounting filesystems](https://minikube.sigs.k8s.io/docs/handbook/mount/) of minikube's official documentation:

> To mount a directory from the host into the guest use the `mount` subcommand

```text
minikube mount $SOURCE_DIR:$MOUNT_PATH
```

```text
📁  Mounting host path /tmp/spark-k8s-demo into VM as /tmp/spark-k8s-demo ...
    ▪ Mount type:
    ▪ User ID:      docker
    ▪ Group ID:     docker
    ▪ Version:      9p2000.L
    ▪ Message Size: 262144
    ▪ Permissions:  755 (-rwxr-xr-x)
    ▪ Options:      map[]
    ▪ Bind Address: 127.0.0.1:53819
🚀  Userspace file server: ufs starting
✅  Successfully mounted /tmp/spark-k8s-demo to /tmp/spark-k8s-demo

📌  NOTE: This process must stay alive for the mount to be accessible ...
```

### Validating Mount

Use `minikube ssh` to validate the volume on the minikube's VM.

```text
minikube ssh
```

```text
docker@minikube:~$ ls -ld /tmp/spark-k8s-demo
drwxr-xr-x 1 docker docker 96 Mar  9 14:11 /tmp/spark-k8s-demo
```

## Using Kubernetes Volumes

Quoting [Using Kubernetes Volumes]({{ spark.doc }}/latest/running-on-kubernetes.html#using-kubernetes-volumes) of Apache Spark's official documentation:

> users can mount the following types of Kubernetes volumes into the driver and executor pods:
>
> * hostPath: mounts a file or directory from the host node’s filesystem into a pod.
> * emptyDir: an initially empty volume created when a pod is assigned to a node.
> * persistentVolumeClaim: used to mount a PersistentVolume into a pod.

### hostPath

Let's use Kubernetes' [hostPath]({{ k8s.doc }}/concepts/storage/volumes/#hostpath) that requires `spark.kubernetes.*.volumes`-prefixed configuration properties for the driver and executor pods:

```text
--conf spark.kubernetes.driver.volumes.$VOLUME_TYPE.$VOLUME_NAME.mount.path=$MOUNT_PATH
--conf spark.kubernetes.driver.volumes.$VOLUME_TYPE.$VOLUME_NAME.options.path=$MOUNT_PATH
```

The demo uses configuration properties to set up a `hostPath` volume type (`$VOLUME_TYPE`) with `$VOLUME_NAME` name and `$MOUNT_PATH` path on the host (for the driver and executors separately).

```text
cd $SPARK_HOME
```

The idea is to let `spark-submit` upload the `--files` to `spark.kubernetes.file.upload.path` directory that is available under the same directory (logically by name as on the host). That's why the names of the source and target mounted directories are the same.

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --files test.me \
  --conf spark.kubernetes.file.upload.path=$SOURCE_DIR \
  --conf spark.kubernetes.driver.volumes.$VOLUME_TYPE.$VOLUME_NAME.mount.path=$MOUNT_PATH \
  --conf spark.kubernetes.driver.volumes.$VOLUME_TYPE.$VOLUME_NAME.options.path=$MOUNT_PATH \
  --conf spark.kubernetes.executor.volumes.$VOLUME_TYPE.$VOLUME_NAME.mount.path=$MOUNT_PATH \
  --conf spark.kubernetes.executor.volumes.$VOLUME_TYPE.$VOLUME_NAME.options.path=$MOUNT_PATH \
  --name $POD_NAME \
  --class meetup.SparkApp \
  --conf spark.kubernetes.container.image=$IMAGE_NAME \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/spark/jars/meetup.meetup-spark-app-0.1.0.jar
```

## Reviewing Volumes

### Describe Pod

```text
k describe po $POD_NAME
```

### Pod Volumes

```text
k get po $POD_NAME -o=jsonpath='{.spec.volumes}' | jq
```

```text
[
  {
    "hostPath": {
      "path": "/tmp/spark-k8s-demo",
      "type": ""
    },
    "name": "demo-host-mount"
  },
  {
    "emptyDir": {},
    "name": "spark-local-dir-1"
  },
  {
    "configMap": {
      "defaultMode": 420,
      "items": [
        {
          "key": "log4j.properties",
          "mode": 420,
          "path": "log4j.properties"
        },
        {
          "key": "spark.properties",
          "mode": 420,
          "path": "spark.properties"
        }
      ],
      "name": "spark-drv-757769781753ba14-conf-map"
    },
    "name": "spark-conf-volume-driver"
  },
  {
    "name": "spark-token-kzmdd",
    "secret": {
      "defaultMode": 420,
      "secretName": "spark-token-kzmdd"
    }
  }
]
```

### Volume Mounts

```text
k get po $POD_NAME -o=jsonpath='{.spec.containers[].volumeMounts}' | jq
```

```text
[
  {
    "mountPath": "/tmp/spark-k8s-demo",
    "name": "demo-host-mount"
  },
  {
    "mountPath": "/var/data/spark-67727eb6-3ca5-4e72-8ea2-20179aca2831",
    "name": "spark-local-dir-1"
  },
  {
    "mountPath": "/opt/spark/conf",
    "name": "spark-conf-volume-driver"
  },
  {
    "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
    "name": "spark-token-kzmdd",
    "readOnly": true
  }
]
```

### Inside Pod

```text
k exec $POD_NAME -- ls -ltr /tmp/
```

```text
$ cat /tmp/spark-k8s/spark-upload-5ded6fb9-b5e7-4a09-9d0f-d3c8c85add08/test.me
Hello World
```
