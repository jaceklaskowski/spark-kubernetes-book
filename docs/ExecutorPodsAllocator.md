# ExecutorPodsAllocator

`ExecutorPodsAllocator` is responsible for [allocating pods for executors](#onNewSnapshots) (possibly [dynamic](#dynamicAllocationEnabled)) in a Spark application.

`ExecutorPodsAllocator` is used to create a [KubernetesClusterSchedulerBackend](KubernetesClusterSchedulerBackend.md#podAllocator).

## Creating Instance

`ExecutorPodsAllocator` takes the following to be created:

* <span id="conf"> `SparkConf`
* <span id="secMgr"> `SecurityManager`
* <span id="executorBuilder"> [KubernetesExecutorBuilder](KubernetesExecutorBuilder.md)
* <span id="kubernetesClient"> `KubernetesClient`
* <span id="snapshotsStore"> [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md)
* <span id="clock"> `Clock`

`ExecutorPodsAllocator` is created when:

* `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)

## <span id="dynamicAllocationEnabled"> spark.dynamicAllocation.enabled

`ExecutorPodsAllocator` uses `spark.dynamicAllocation.enabled` configuration property to turn dynamic allocation of executors on and off.

!!! tip "The Internals of Apache Spark"
    Learn more about [Dynamic Allocation of Executors]({{ book.spark_core }}/dynamic-allocation/) in [The Internals of Apache Spark]({{ book.spark_core }}).

## <span id="driverPod"> Driver Pod

```scala
driverPod: Option[Pod]
```

`driverPod` is a driver pod with the name of [spark.kubernetes.driver.pod.name](#kubernetesDriverPodName) configuration property (if defined).

`ExecutorPodsAllocator` throws a `SparkException` when the driver pod could not be found in a Kubernetes cluster:

```text
No pod was found named [kubernetesDriverPodName] in the cluster in the namespace [namespace] (this was supposed to be the driver pod.).
```

### <span id="kubernetesDriverPodName"> spark.kubernetes.driver.pod.name

`ExecutorPodsAllocator` uses [spark.kubernetes.driver.pod.name](configuration-properties.md#spark.kubernetes.driver.pod.name) configuration property to [look up the driver pod by name](#driverPod) when [created](#creating-instance).

## <span id="podAllocationSize"> spark.kubernetes.allocation.batch.size

`ExecutorPodsAllocator` uses [spark.kubernetes.allocation.batch.size](configuration-properties.md#spark.kubernetes.allocation.batch.size) configuration property in the following:

* [onNewSnapshots](#onNewSnapshots)

## <span id="podAllocationDelay"> spark.kubernetes.allocation.batch.delay

`ExecutorPodsAllocator` uses [spark.kubernetes.allocation.batch.delay](configuration-properties.md#spark.kubernetes.allocation.batch.delay) configuration property for the following:

* [podCreationTimeout](#podCreationTimeout)
* [Registering a subscriber](#start)

## <span id="shouldDeleteExecutors"> spark.kubernetes.executor.deleteOnTermination

`ExecutorPodsAllocator` uses [spark.kubernetes.executor.deleteOnTermination](configuration-properties.md#spark.kubernetes.executor.deleteOnTermination) configuration property.

## <span id="start"> Starting

```scala
start(
  applicationId: String): Unit
```

`start` requests the [ExecutorPodsSnapshotsStore](#snapshotsStore) to [subscribe](ExecutorPodsSnapshotsStore.md#addSubscriber) this `ExecutorPodsAllocator` to [be notified about new snapshots](#onNewSnapshots) (with pod allocation delay based on [spark.kubernetes.allocation.batch.delay](configuration-properties.md#spark.kubernetes.allocation.batch.delay) configuration property).

`start` is used when:

* `KubernetesClusterSchedulerBackend` is requested to [start](KubernetesClusterSchedulerBackend.md#start)

## <span id="onNewSnapshots"> Processing Executor Pods Snapshots

```scala
onNewSnapshots(
  applicationId: String,
  snapshots: Seq[ExecutorPodsSnapshot]): Unit
```

`onNewSnapshots` removes the executor IDs (of the executor pods in the given snapshots) from the [newlyCreatedExecutors](#newlyCreatedExecutors) internal registry.

`onNewSnapshots` finds timed-out executor IDs (in the [newlyCreatedExecutors](#newlyCreatedExecutors) internal registry) whose creation time exceeded some [podCreationTimeout](#podCreationTimeout) threshold. For the other executor IDs, `onNewSnapshots` prints out the following DEBUG message to the logs:

```text
Executor with id [execId] was not found in the Kubernetes cluster since it was created [time] milliseconds ago.
```

For any timed-out executor IDs, `onNewSnapshots` prints out the following WARN message to the logs:

```text
Executors with ids [ids] were not detected in the Kubernetes cluster after [podCreationTimeout] ms despite the fact that a previous allocation attempt tried to create them. The executors may have been deleted but the application missed the deletion event.
```

`onNewSnapshots` removes (_forgets_) the timed-out executor IDs (from the [newlyCreatedExecutors](#newlyCreatedExecutors) internal registry). With the [shouldDeleteExecutors](#shouldDeleteExecutors) flag enabled, `onNewSnapshots` requests the [KubernetesClient](#kubernetesClient) to delete pods with the following labels:

* `spark-app-selector` with the given `applicationId`
* `spark-role`=`executor`
* `spark-exec-id` for all timed-out executor IDs

`onNewSnapshots` updates the [lastSnapshot](#lastSnapshot) internal registry with the last `ExecutorPodsSnapshot` among the given `snapshots` if available.

`onNewSnapshots` counts running executor pods in the [lastSnapshot](#lastSnapshot) internal registry.

`onNewSnapshots` finds pending executor IDs in the [lastSnapshot](#lastSnapshot) internal registry.

For non-empty input `snapshots`, `onNewSnapshots` prints out the following DEBUG message to the logs:

```text
Pod allocation status: [currentRunningCount] running, [currentPendingExecutors] pending, [newlyCreatedExecutors] unacknowledged.
```

`onNewSnapshots`...FIXME

In the end, with DEBUG logging enabled or the input `snapshots` is empty, `onNewSnapshots` prints out the following DEBUG messages.

With the number of the executor pods currently running higher than the [total expected executors](#totalExpectedExecutors) but no [dynamicAllocationEnabled](#dynamicAllocationEnabled), `onNewSnapshots` prints out the following:

```text
Current number of running executors is equal to the number of requested executors. Not scaling up further.
```

Otherwise, when there are executor pods pending (_outstanding_), `onNewSnapshots` prints out the following:

```text
Still waiting for [outstanding] executors before requesting more.
```

## <span id="totalExpectedExecutors"> Total Expected Executors

```scala
totalExpectedExecutors: AtomicInteger
```

`ExecutorPodsAllocator` uses a Java [AtomicInteger]({{ java.api }}/java.base/java/util/concurrent/atomic/AtomicInteger.html) to track the total expected number of executors.

Starts from `0` and is set to a fixed number of the total expected executors in [setTotalExpectedExecutors](#setTotalExpectedExecutors)

Used in [onNewSnapshots](#onNewSnapshots)

### <span id="setTotalExpectedExecutors"> Changing Total Expected Executors

```scala
setTotalExpectedExecutors(
  total: Int): Unit
```

`setTotalExpectedExecutors` sets [totalExpectedExecutors](#totalExpectedExecutors) internal registry to the input `total`.

With no [hasPendingPods](#hasPendingPods), `setTotalExpectedExecutors` requests the [ExecutorPodsSnapshotsStore](#snapshotsStore) to [notifySubscribers](ExecutorPodsSnapshotsStore.md#notifySubscribers).

`setTotalExpectedExecutors` is used when:

* `KubernetesClusterSchedulerBackend` is requested to [start](KubernetesClusterSchedulerBackend.md#start) and [doRequestTotalExecutors](KubernetesClusterSchedulerBackend.md#doRequestTotalExecutors)

## Registries

### <span id="newlyCreatedExecutors"> newlyCreatedExecutors

```scala
newlyCreatedExecutors: Map[Long, Long]
```

`ExecutorPodsAllocator` uses `newlyCreatedExecutors` internal registry to track executor IDs (with the timestamps they were created) that have been requested from Kubernetes but have not been detected in any snapshot yet.

Used in [onNewSnapshots](#onNewSnapshots)

### <span id="EXECUTOR_ID_COUNTER"> EXECUTOR_ID_COUNTER

`ExecutorPodsAllocator` uses a Java [AtomicLong]({{ java.api }}/java.base/java/util/concurrent/atomic/AtomicLong.html) for the missing executor IDs that are going to be requested (in [onNewSnapshots](#onNewSnapshots)) when the number of running executor pods is below the [total expected executors](#totalExpectedExecutors).

### <span id="hasPendingPods"> hasPendingPods Flag

```scala
hasPendingPods: AtomicBoolean
```

`ExecutorPodsAllocator` uses a Java [AtomicBoolean]({{ java.api }}/java.base/java/util/concurrent/atomic/AtomicBoolean.html) as a flag to avoid notifying subscribers.

Starts as `false` and is updated every [onNewSnapshots](#onNewSnapshots)

Used in [setTotalExpectedExecutors](#setTotalExpectedExecutors) (only when `false`)

## Logging

Enable `ALL` logging level for `org.apache.spark.scheduler.cluster.k8s.ExecutorPodsAllocator` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```text
log4j.logger.org.apache.spark.scheduler.cluster.k8s.ExecutorPodsAllocator=ALL
```

Refer to [Logging](spark-logging.md).
