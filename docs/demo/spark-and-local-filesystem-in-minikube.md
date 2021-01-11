---
hide:
  - navigation
---

# Demo: Spark and Local Filesystem in minikube

The motivation of the demo is to set up a Spark application deployed to minikube to access files on a local filesystem.

!!! tip
    Start with [Demo: Running Spark Application on minikube](running-spark-application-on-minikube.md).

The demo uses `spark-submit --files` and [spark.kubernetes.file.upload.path](../configuration-properties.md#spark.kubernetes.file.upload.path) configuration property to upload a static file to a directory that is then mounted to Spark application pods.

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --files test.me --conf spark.kubernetes.file.upload.path=/tmp/spark-k8s \
  --name spark-docker-example \
  --class meetup.SparkApp \
  --conf spark.kubernetes.container.image=spark-docker-example:0.1.0 \
  --conf spark.kubernetes.driver.pod.name=spark-demo-minikube \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/docker/lib/meetup.spark-docker-example-0.1.0.jar
```

## Mounting Filesystems

Let's start off by mounting a host directory (`/tmp/spark-k8s`) to minikube.

Quoting [Mounting filesystems](https://minikube.sigs.k8s.io/docs/handbook/mount/) of minikube's official documentation:

> To mount a directory from the host into the guest using the `mount` subcommand

```text
$ minikube mount /tmp/spark-k8s:/tmp/spark-k8s
📁  Mounting host path /tmp/spark-k8s into VM as /tmp/spark-k8s ...
    ▪ Mount type:
    ▪ User ID:      docker
    ▪ Group ID:     docker
    ▪ Version:      9p2000.L
    ▪ Message Size: 262144
    ▪ Permissions:  755 (-rwxr-xr-x)
    ▪ Options:      map[]
    ▪ Bind Address: 127.0.0.1:53125
🚀  Userspace file server: ufs starting
✅  Successfully mounted /tmp/spark-k8s to /tmp/spark-k8s

📌  NOTE: This process must stay alive for the mount to be accessible ...
```

## Using Kubernetes Volumes

Quoting [Using Kubernetes Volumes]({{ spark.doc }}/latest/running-on-kubernetes.html#using-kubernetes-volumes) of Apache Spark's official documentation:

> users can mount the following types of Kubernetes volumes into the driver and executor pods:
>
> * hostPath: mounts a file or directory from the host node’s filesystem into a pod.
> * emptyDir: an initially empty volume created when a pod is assigned to a node.
> * persistentVolumeClaim: used to mount a PersistentVolume into a pod.

### hostPath

Let's use Kubernetes' [hostPath]({{ k8s.doc }}/concepts/storage/volumes/#hostpath) that requires `spark.kubernetes.*.volumes`-prefixed configuration properties and the name of the volume (under the `volumes` field in the pod specification):

```text
--conf spark.kubernetes.driver.volumes.hostPath.spark-k8s.mount.path=/tmp/spark-k8s
```

The demo uses configuration properties to set up a `hostPath` volume type with `spark-k8s` name and `/tmp/spark-k8s` path on the host (for the driver and executors separately).

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --files test.me --conf spark.kubernetes.file.upload.path=/tmp/spark-k8s \
  --conf spark.kubernetes.driver.volumes.hostPath.spark-k8s.mount.path=/tmp/spark-k8s \
  --conf spark.kubernetes.driver.volumes.hostPath.spark-k8s.options.path=/tmp/spark-k8s \
  --conf spark.kubernetes.executor.volumes.hostPath.spark-k8s.mount.path=/tmp/spark-k8s \
  --conf spark.kubernetes.executor.volumes.hostPath.spark-k8s.options.path=/tmp/spark-k8s \
  --name spark-docker-example \
  --class meetup.SparkApp \
  --conf spark.kubernetes.container.image=spark-docker-example:0.1.0 \
  --conf spark.kubernetes.driver.pod.name=spark-demo-minikube \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/docker/lib/meetup.spark-docker-example-0.1.0.jar
```

## Reviewing Volumes

```text
k describe po spark-demo-minikube
```

```text
$ k get po spark-demo-minikube -o=jsonpath='{.spec.volumes}' | jq
[
  {
    "hostPath": {
      "path": "/tmp/spark-k8s",
      "type": ""
    },
    "name": "spark-k8s"
  },
  {
    "emptyDir": {},
    "name": "spark-local-dir-1"
  },
  {
    "configMap": {
      "defaultMode": 420,
      "name": "spark-docker-example-85c0cb76f277cdbe-driver-conf-map"
    },
    "name": "spark-conf-volume"
  },
  {
    "name": "spark-token-zd6jt",
    "secret": {
      "defaultMode": 420,
      "secretName": "spark-token-zd6jt"
    }
  }
]
```

```text
$ k get po spark-demo-minikube -o=jsonpath='{.spec.containers[].volumeMounts}' | jq
[
  {
    "mountPath": "/tmp/spark-k8s",
    "name": "spark-k8s"
  },
  {
    "mountPath": "/var/data/spark-87d1ba7c-819a-4274-82ca-98c1e135c136",
    "name": "spark-local-dir-1"
  },
  {
    "mountPath": "/opt/spark/conf",
    "name": "spark-conf-volume"
  },
  {
    "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
    "name": "spark-token-zd6jt",
    "readOnly": true
  }
]
```

Log into the driver pod container and review `/tmp/spark-k8s` directory. There should be at least one `spark-upload` directory.

```text
$ k exec -it spark-demo-minikube -- bash

$ ls -ltr /tmp/spark-k8s
total 5
...
drwxr-xr-x 1 1000 999 96 Jan 11 17:21 spark-upload-5ded6fb9-b5e7-4a09-9d0f-d3c8c85add08

$ cat /tmp/spark-k8s/spark-upload-5ded6fb9-b5e7-4a09-9d0f-d3c8c85add08/test.me
Hello World
```
