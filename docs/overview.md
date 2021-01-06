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

## Demo

1. [spark-shell on minikube](demo/spark-shell-on-minikube.md)
1. [Running Spark Application on minikube](demo/running-spark-application-on-minikube.md)

## Resources

* [Official documentation]({{ spark.doc }}/running-on-kubernetes.html)
* [Spark on Kubernetes](https://levelup.gitconnected.com/spark-on-kubernetes-3d822969f85b) by Scott Haines
* (video) [Getting Started with Apache Spark on Kubernetes](https://www.youtube.com/watch?v=xo7BIkFWQP4) by Jean-Yves Stephan and Julien Dumazert
