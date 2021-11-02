# {{ book.title }}

[Kubernetes](https://kubernetes.io/) is an open-source system for automating deployment, scaling, and management of containerized applications.

Apache Spark supports `Kubernetes` resource manager using [KubernetesClusterManager](KubernetesClusterManager.md) (and [KubernetesClusterSchedulerBackend](KubernetesClusterSchedulerBackend.md)) with **k8s://**-prefixed master URLs (that point at [Kubernetes API servers](https://kubernetes.io/docs/concepts/overview/components/#kube-apiserver)).

Spark on Kubernetes uses `TaskSchedulerImpl` ([Apache Spark]({{ book.spark_core }}/scheduler/TaskSchedulerImpl/)) for task scheduling.

## <span id="SPARK_EXECUTOR_INACTIVE_LABEL"> Inactive Executor Pods

Spark on Kubernetes defines **spark-exec-inactive** label to mark executor pods as inactive after they have finished (successfully or not) but [spark.kubernetes.executor.deleteOnTermination](configuration-properties.md#spark.kubernetes.executor.deleteOnTermination) configuration property is `false` (when `ExecutorPodsLifecycleManager` is requested to [handle executor pods snapshots](ExecutorPodsLifecycleManager.md#onNewSnapshots)).

This label is used to skip executor pods when `PollRunnable` is requested to [fetch status of all executor pods in a Spark application from Kubernetes API server](PollRunnable.md#run).

## Cluster Deploy Mode

Spark on Kubernetes uses [KubernetesClientApplication](KubernetesClientApplication.md) in `cluster` deploy mode (as the `SparkApplication` ([Apache Spark]({{ book.spark_core }}/tools/SparkApplication/)) to run).

!!! note
    Use `spark-submit --deploy-mode`, `spark.submit.deployMode` or `DEPLOY_MODE` environment variable to specify the deploy mode of a Spark application.

## Volumes

Volumes and volume mounts are configured using `spark.kubernetes.[type].volumes.`-prefixed configuration properties with `type` being `driver` or `executor` (for the driver and executor pods, respectively).

`KubernetesVolumeUtils` utility is used to [extract volume configuration](KubernetesVolumeUtils.md#parseVolumeSpecificConf) based on the volume type:

Volume Type  | Configuration Property
-------------|---------
 `emptyDir`  | `[volumesPrefix].[volumeType].[volumeName].options.medium`
  &nbsp;     | `[volumesPrefix].[volumeType].[volumeName].options.sizeLimit`
 `hostPath`  | `[volumesPrefix].[volumeType].[volumeName].options.path`
 `persistentVolumeClaim` | `[volumesPrefix].[volumeType].[volumeName].options.claimName`

Executor volumes (`spark.kubernetes.executor.volumes.`-prefixed configuration properties) are parsed right when `KubernetesConf` utility is used for a [KubernetesDriverConf](KubernetesConf.md#createDriverConf) (and a driver pod created). That makes executor volumes required when driver volumes are defined.

## Static File Resources

**File resources** are resources with `file` or no URI scheme (that are then considered `file`-based indirectly).

In Spark applications, file resources can be the primary resource (application jar, Python or R files) as well as files referenced by `spark.jars` and `spark.files` configuration properties (or their `--jars` and `--files` options of `spark-submit`, respectively).

When deployed in `cluster` mode, Spark on Kubernetes uploads file resources of a Spark application to a Hadoop DFS-compatible file system defined by the required [spark.kubernetes.file.upload.path](configuration-properties.md#spark.kubernetes.file.upload.path) configuration property.

### Local URI Scheme

A special case of static file resources are **local resources** that are resources with `local` URI scheme. They are considered already available on every Spark node (and are not added to a Spark file server for distribution when `SparkContext` is requested to [add such file]({{ book.spark_core }}/SparkContext/#static-files)).

In Spark on Kubernetes, `local` resources are used for primary application resource that are already included in a container image.

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  local:///opt/docker/lib/meetup.spark-docker-example-0.1.0.jar
```

## Executor Pods State Synchronization

Spark on Kubernetes uses [ExecutorPodsPollingSnapshotSource](ExecutorPodsPollingSnapshotSource.md) for [polling Kubernetes API server for executor pods of a Spark application](PollRunnable.md#run) every polling interval (based on [spark.kubernetes.executor.apiPollingInterval](configuration-properties.md#spark.kubernetes.executor.apiPollingInterval) configuration property).

`ExecutorPodsPollingSnapshotSource` is given an [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md) that is requested to [replace a snapshot](ExecutorPodsSnapshotsStore.md#replaceSnapshot) regularly.

`ExecutorPodsSnapshotsStore` keeps track of executor pods state snapshots and allows [subscribers](ExecutorPodsSnapshotsStore.md#addSubscriber) to be regularly updated (e.g. [ExecutorPodsAllocator](ExecutorPodsAllocator.md) and [ExecutorPodsLifecycleManager](ExecutorPodsLifecycleManager.md)).

## Dynamic Allocation of Executors

Spark on Kubernetes supports **Dynamic Allocation of Executors** using [ExecutorPodsAllocator](ExecutorPodsAllocator.md).

!!! tip "The Internals of Apache Spark"
    Learn more about [Dynamic Allocation of Executors]({{ book.spark_core }}/dynamic-allocation/) in [The Internals of Apache Spark]({{ book.spark_core }}).

## <span id="spark-internal"> Internal Resource Marker

Spark on Kubernetes uses **spark-internal** special name in `cluster` deploy mode for internal application resources (that are supposed to be part of an image).

Given [renameMainAppResource](KubernetesUtils.md#renameMainAppResource), `DriverCommandFeatureStep` will re-write local `file`-scheme-based primary application resources to `spark-internal` special name when requested for the [base driver container](DriverCommandFeatureStep.md#baseDriverContainer) (for a `JavaMainAppResource` application).

### <span id="spark-internal-demo"> Demo

This demo is a follow-up to [Demo: Running Spark Application on minikube](demo/running-spark-application-on-minikube.md). Run it first.

Note `--deploy-mode cluster` and the application jar is "locally resolvable" (i.e. uses `file:` scheme indirectly).

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name spark-docker-example \
  --class meetup.SparkApp \
  --conf spark.kubernetes.container.image=spark-docker-example:0.1.0 \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --conf spark.kubernetes.file.upload.path=/tmp/spark-k8s \
  --verbose \
  ~/dev/meetups/spark-meetup/spark-docker-example/target/scala-2.12/spark-docker-example_2.12-0.1.0.jar
```

```text
$ kubectl get po -l spark-role=driver
NAME                                           READY   STATUS   RESTARTS   AGE
spark-docker-example-dfd7d076e7099718-driver   0/1     Error    0          7m25s
```

Note `spark-internal` in the below output.

```text
$ kubectl describe po spark-docker-example-dfd7d076e7099718-driver
...
Containers:
  spark-kubernetes-driver:
    ...
    Args:
      driver
      --properties-file
      /opt/spark/conf/spark.properties
      --class
      meetup.SparkApp
      spark-internal
...
```

## Resources

* [Official documentation]({{ spark.doc }}/running-on-kubernetes.html)
* [SPARK-33005 Kubernetes GA Preparation](https://issues.apache.org/jira/browse/SPARK-33005)
* [Spark on Kubernetes](https://levelup.gitconnected.com/spark-on-kubernetes-3d822969f85b) by Scott Haines
* (video) [Getting Started with Apache Spark on Kubernetes](https://www.youtube.com/watch?v=xo7BIkFWQP4) by Jean-Yves Stephan and Julien Dumazert
