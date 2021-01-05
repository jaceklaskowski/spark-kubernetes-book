# KubernetesClusterSchedulerBackend

`KubernetesClusterSchedulerBackend` is a `CoarseGrainedSchedulerBackend` ([Apache Spark]({{ book.spark_core }}/scheduler/CoarseGrainedSchedulerBackend/)) for [Spark on Kubernetes](index.md).

## Creating Instance

`KubernetesClusterSchedulerBackend` takes the following to be created:

* <span id="scheduler"> `TaskSchedulerImpl` ([Apache Spark]({{ book.spark_core }}/scheduler/TaskSchedulerImpl/))
* <span id="sc"> `SparkContext` ([Apache Spark]({{ book.spark_core }}/SparkContext/))
* <span id="kubernetesClient"> `KubernetesClient`
* <span id="executorService"> Java's [ScheduledExecutorService]({{ java.api }}/java.base/java/util/concurrent/ScheduledExecutorService.html)
* <span id="snapshotsStore"> [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md)
* [ExecutorPodsAllocator](#podAllocator)
* [ExecutorPodsLifecycleManager](#lifecycleEventHandler)
* <span id="watchEvents"> [ExecutorPodsWatchSnapshotSource](ExecutorPodsWatchSnapshotSource.md)
* <span id="pollEvents"> [ExecutorPodsPollingSnapshotSource](ExecutorPodsPollingSnapshotSource.md)

`KubernetesClusterSchedulerBackend` is created when:

* `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)

## <span id="lifecycleEventHandler"><span id="ExecutorPodsLifecycleManager"> ExecutorPodsLifecycleManager

`KubernetesClusterSchedulerBackend` is given an [ExecutorPodsLifecycleManager](ExecutorPodsLifecycleManager.md) to be [created](#creating-instance).

`KubernetesClusterSchedulerBackend` requests the `ExecutorPodsLifecycleManager` to [start](ExecutorPodsLifecycleManager.md#start) (with itself) when [started](#start).

## <span id="podAllocator"><span id="ExecutorPodsAllocator"> ExecutorPodsAllocator

`KubernetesClusterSchedulerBackend` is given an [ExecutorPodsAllocator](ExecutorPodsAllocator.md) to be [created](#creating-instance).

When [started](#start), `KubernetesClusterSchedulerBackend` requests the `ExecutorPodsAllocator` to [setTotalExpectedExecutors](ExecutorPodsAllocator.md#setTotalExpectedExecutors) to the [number of initial executors](#initialExecutors) and [starts it](ExecutorPodsAllocator.md#start) with [application Id](#applicationId).

When requested for the [expected number of executors](#doRequestTotalExecutors), `KubernetesClusterSchedulerBackend` requests the `ExecutorPodsAllocator` to [setTotalExpectedExecutors](ExecutorPodsAllocator.md#setTotalExpectedExecutors) to the given total number of executors.

When requested to [isBlacklisted](#isBlacklisted), `KubernetesClusterSchedulerBackend` requests the `ExecutorPodsAllocator` to [isDeleted](ExecutorPodsAllocator.md#isDeleted) with a given executor.

## <span id="initialExecutors"> Initial Executors

```scala
initialExecutors: Int
```

`KubernetesClusterSchedulerBackend` calculates the initial target number of executors when [created](#creating-instance).

`initialExecutors` is used when `KubernetesClusterSchedulerBackend` is requested to [start](#start) and [whether or not sufficient resources registered](#sufficientResourcesRegistered).

## <span id="applicationId"> Application Id

```scala
applicationId(): String
```

`applicationId` is part of the `SchedulerBackend` ([Apache Spark]({{ book.spark_core }}/scheduler/SchedulerBackend#applicationId)) abstraction.

`applicationId` is the value of `spark.app.id` configuration property if defined or the default `applicationId`.

## <span id="sufficientResourcesRegistered"> Sufficient Resources Registered

```scala
sufficientResourcesRegistered(): Boolean
```

`sufficientResourcesRegistered` is part of the `CoarseGrainedSchedulerBackend` ([Apache Spark]({{ book.spark_core }}/scheduler/CoarseGrainedSchedulerBackend#sufficientResourcesRegistered)) abstraction.

`sufficientResourcesRegistered` holds (is `true`) when the `totalRegisteredExecutors` is at least the [ratio](#minRegisteredRatio) of the [initial executors](#initialExecutors).

## <span id="minRegisteredRatio"> Minimum Resources Available Ratio

```scala
minRegisteredRatio: Double
```

`minRegisteredRatio` is part of the `CoarseGrainedSchedulerBackend` ([Apache Spark]({{ book.spark_core }}/scheduler/CoarseGrainedSchedulerBackend#minRegisteredRatio)) abstraction.

`minRegisteredRatio` is `0.8` unless `spark.scheduler.minRegisteredResourcesRatio` configuration property is defined.

## <span id="start"> Starting SchedulerBackend

```scala
start(): Unit
```

`start` is part of the `CoarseGrainedSchedulerBackend` ([Apache Spark]({{ book.spark_core }}/scheduler/CoarseGrainedSchedulerBackend#start)) abstraction.

`start` creates a delegation token manager.

`start` requests the [ExecutorPodsAllocator](#podAllocator) to [setTotalExpectedExecutors](ExecutorPodsAllocator.md#setTotalExpectedExecutors) to [initialExecutors](#initialExecutors).

`start` requests the [ExecutorPodsLifecycleManager](#lifecycleEventHandler) to [start](ExecutorPodsLifecycleManager.md#start) (with this `KubernetesClusterSchedulerBackend`).

`start` requests the [ExecutorPodsAllocator](#podAllocator) to [start](ExecutorPodsAllocator.md#start) (with the [applicationId](#applicationId))

`start` requests the [ExecutorPodsWatchSnapshotSource](#watchEvents) to [start](ExecutorPodsWatchSnapshotSource.md#start) (with the [applicationId](#applicationId))

`start` requests the [ExecutorPodsPollingSnapshotSource](#pollEvents) to [start](ExecutorPodsPollingSnapshotSource.md#start) (with the [applicationId](#applicationId))

## <span id="createDriverEndpoint"> Creating DriverEndpoint

```scala
createDriverEndpoint(): DriverEndpoint
```

`createDriverEndpoint` is part of the `CoarseGrainedSchedulerBackend` ([Apache Spark]({{ book.spark_core }}/scheduler/CoarseGrainedSchedulerBackend#createDriverEndpoint)) abstraction.

`createDriverEndpoint` creates a [KubernetesDriverEndpoint](KubernetesDriverEndpoint.md).

## <span id="doRequestTotalExecutors"> Requesting Executors from Cluster Manager

```scala
doRequestTotalExecutors(
  requestedTotal: Int): Future[Boolean]
```

`doRequestTotalExecutors` is part of the `CoarseGrainedSchedulerBackend` ([Apache Spark]({{ book.spark_core }}/scheduler/CoarseGrainedSchedulerBackend#doRequestTotalExecutors)) abstraction.

`doRequestTotalExecutors` requests the [ExecutorPodsAllocator](#podAllocator) to [setTotalExpectedExecutors](ExecutorPodsAllocator.md#setTotalExpectedExecutors) to the given `requestedTotal`.

In the end, `doRequestTotalExecutors` returns a completed `Future` with `true` value.

## <span id="stop"> Stopping SchedulerBackend

```scala
stop(): Unit
```

`stop` is part of the `CoarseGrainedSchedulerBackend` ([Apache Spark]({{ book.spark_core }}/scheduler/CoarseGrainedSchedulerBackend#stop)) abstraction.

`stop`...FIXME
