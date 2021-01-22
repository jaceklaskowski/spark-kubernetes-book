# ExecutorPodsSnapshotsStoreImpl

`ExecutorPodsSnapshotsStoreImpl` is an [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md).

## Creating Instance

`ExecutorPodsSnapshotsStoreImpl` takes the following to be created:

* <span id="subscribersExecutor"> Java's [ScheduledExecutorService]({{ java.api }}/java.base/java/util/concurrent/ScheduledExecutorService.html)

`ExecutorPodsSnapshotsStoreImpl` is created when:

* `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)

## <span id="replaceSnapshot"> replaceSnapshot

```scala
replaceSnapshot(
  newSnapshot: Seq[Pod]): Unit
```

`replaceSnapshot` is part of the [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md#replaceSnapshot) abstraction.

`replaceSnapshot` replaces the [currentSnapshot](#currentSnapshot) internal registry with a new [ExecutorPodsSnapshot](ExecutorPodsSnapshot.md) (with the given snapshot and the current time).

In the end, `updatePod` [addCurrentSnapshotToSubscribers](#addCurrentSnapshotToSubscribers).

## <span id="updatePod"> updatePod

```scala
updatePod(
  updatedPod: Pod): Unit
```

`updatePod` is part of the [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md#updatePod) abstraction.

`updatePod` requests the [current ExecutorPodsSnapshot](#currentSnapshot) to [update](ExecutorPodsSnapshot.md#withUpdate) based on the given updated pod.

In the end, `updatePod` [addCurrentSnapshotToSubscribers](#addCurrentSnapshotToSubscribers).

## <span id="addSubscriber"> Registering Subscriber

```scala
addSubscriber(
  processBatchIntervalMillis: Long)
  (onNewSnapshots: Seq[ExecutorPodsSnapshot] => Unit): Unit
```

`addSubscriber` is part of the [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md#addSubscriber) abstraction.

`addSubscriber` adds a new `SnapshotsSubscriber` to the [subscribers](#subscribers) internal registry.

`addSubscriber` requests the [ScheduledExecutorService](#subscribersExecutor) to schedule [processing executor pods](#callSubscriber) by the `SnapshotsSubscriber` every given `processBatchIntervalMillis` delay (starting immediately).

In the end, `addSubscriber` adds the scheduled action to the [pollingTasks](#pollingTasks) internal registry.

## <span id="callSubscriber"> callSubscriber

```scala
callSubscriber(
  subscriber: SnapshotsSubscriber): Unit
```

`callSubscriber`...FIXME

`callSubscriber` is used when:

* `ExecutorPodsSnapshotsStoreImpl` is requested to [addSubscriber](#addSubscriber) and [notifySubscribers](#notifySubscribers)

## <span id="pollingTasks"> pollingTasks Registry

```scala
pollingTasks: CopyOnWriteArrayList[Future[_]]
```

`ExecutorPodsSnapshotsStoreImpl` uses `pollingTasks` internal registry to track the recurring actions scheduled for [subscribers](#subscribers).

`pollingTasks` are cancelled when `ExecutorPodsSnapshotsStoreImpl` is requested to [stop](#stop).

## <span id="subscribers"> subscribers Registry

```scala
subscribers: CopyOnWriteArrayList[SnapshotsSubscriber]
```

`ExecutorPodsSnapshotsStoreImpl` uses `subscribers` internal registry to track [subscribers](#addSubscriber) that want to be notified regularly about the current state of executor pods in a cluster.

## <span id="addCurrentSnapshotToSubscribers"> addCurrentSnapshotToSubscribers

```scala
addCurrentSnapshotToSubscribers(): Unit
```

`addCurrentSnapshotToSubscribers` requests every [SnapshotsSubscriber](#subscribers) to [addCurrentSnapshot](SnapshotsSubscriber.md#addCurrentSnapshot).

`addCurrentSnapshotToSubscribers` is used when:

* `ExecutorPodsSnapshotsStoreImpl` is requested to [updatePod](#updatePod) and [replaceSnapshot](#replaceSnapshot)
