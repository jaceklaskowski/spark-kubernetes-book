# ExecutorPodsSnapshotsStore

`ExecutorPodsSnapshotsStore` is an [abstraction](#contract) of [executor pods snapshots stores](#implementations) that subscribers can [subscribe to](#addSubscriber) and be [notified](#notifySubscribers) about [single pod](#updatePod) or [full snapshots updates](#replaceSnapshot).

## Contract

### <span id="addSubscriber"> Registering Subscriber

```scala
addSubscriber(
  processBatchIntervalMillis: Long)(
  onNewSnapshots: Seq[ExecutorPodsSnapshot] => Unit): Unit
```

Registers a new subscriber to be notified about new [ExecutorPodsSnapshot](ExecutorPodsSnapshot.md)s every `processBatchIntervalMillis` interval

Used when:

* `ExecutorPodsAllocator` is requested to [start](ExecutorPodsAllocator.md#start)
* `ExecutorPodsLifecycleManager` is requested to [start](ExecutorPodsLifecycleManager.md#start)

### <span id="notifySubscribers"> Notifying Subscribers

```scala
notifySubscribers(): Unit
```

Used when:

* `ExecutorPodsAllocator` is requested to [change the total expected executors](ExecutorPodsAllocator.md#setTotalExpectedExecutors)

### <span id="replaceSnapshot"> Full Executor Pod State Synchronization

```scala
replaceSnapshot(
  newSnapshot: Seq[Pod]): Unit
```

Used when:

* `PollRunnable` is requested to [start](PollRunnable.md#run) (every [spark.kubernetes.executor.apiPollingInterval](configuration-properties.md#spark.kubernetes.executor.apiPollingInterval) when `ExecutorPodsPollingSnapshotSource` is [started](ExecutorPodsPollingSnapshotSource.md#start))

### <span id="stop"> Stopping

```scala
stop(): Unit
```

Used when:

* `KubernetesClusterSchedulerBackend` is requested to [stop](KubernetesClusterSchedulerBackend.md#stop)

### <span id="updatePod"> Single Executor Pod State Update

```scala
updatePod(
  updatedPod: Pod): Unit
```

Used when:

* `ExecutorPodsWatcher` is requested to [eventReceived](ExecutorPodsWatcher.md#eventReceived)

## Implementations

* [ExecutorPodsSnapshotsStoreImpl](ExecutorPodsSnapshotsStoreImpl.md)
