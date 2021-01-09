# Spark on Kubernetes

[Kubernetes](https://kubernetes.io/) is an open-source system for automating deployment, scaling, and management of containerized applications.

Apache Spark supports `Kubernetes` resource manager as a scheduler using [KubernetesClusterManager](KubernetesClusterManager.md) and [KubernetesClusterSchedulerBackend](KubernetesClusterSchedulerBackend.md) for **k8s://**-prefixed master URLs (that point at [Kubernetes API servers](https://kubernetes.io/docs/concepts/overview/components/#kube-apiserver)).

## Spark 3.1.0

As per [SPARK-33005 Kubernetes GA Preparation](https://issues.apache.org/jira/browse/SPARK-33005), Spark 3.1.0 comes with many improvements for Kubernetes support and is expected to get **General Availability (GA)** marker 🎉

## Executor Pods State Synchronization

Spark on Kubernetes uses [ExecutorPodsPollingSnapshotSource](ExecutorPodsPollingSnapshotSource.md) for polling Kubernetes API server for executor pods state snapshot of a Spark application every polling interval (based on [spark.kubernetes.executor.apiPollingInterval](configuration-properties.md#spark.kubernetes.executor.apiPollingInterval) configuration property).

`ExecutorPodsPollingSnapshotSource` is given an [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md) that is requested to [replaceSnapshot](ExecutorPodsSnapshotsStore.md#replaceSnapshot) regularly.

`ExecutorPodsSnapshotsStore` keeps track of executor pods state snapshots and allows [subscribers](ExecutorPodsSnapshotsStore.md#addSubscriber) to be regularly updated (e.g. [ExecutorPodsAllocator](ExecutorPodsAllocator.md) and [ExecutorPodsLifecycleManager](ExecutorPodsLifecycleManager.md)).

## Dynamic Allocation of Executors

Spark on Kubernetes supports **Dynamic Allocation of Executors** using [ExecutorPodsAllocator](ExecutorPodsAllocator.md).

!!! tip "The Internals of Apache Spark"
    Learn more about [Dynamic Allocation of Executors]({{ book.spark_core }}/dynamic-allocation/) in [The Internals of Apache Spark]({{ book.spark_core }}).

## Local Jars and Files

In `cluster` deploy mode, local files (in `spark.jars` and `spark.files` configuration properties) are uploaded to a Hadoop DFS-compatible file system (defined by [spark.kubernetes.file.upload.path](configuration-properties.md#spark.kubernetes.file.upload.path) configuration property).

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

## Demo

1. [spark-shell on minikube](demo/spark-shell-on-minikube.md)
1. [Running Spark Application on minikube](demo/running-spark-application-on-minikube.md)

## Resources

* [Official documentation]({{ spark.doc }}/running-on-kubernetes.html)
* [Spark on Kubernetes](https://levelup.gitconnected.com/spark-on-kubernetes-3d822969f85b) by Scott Haines
* (video) [Getting Started with Apache Spark on Kubernetes](https://www.youtube.com/watch?v=xo7BIkFWQP4) by Jean-Yves Stephan and Julien Dumazert
