# SnapshotsSubscriber

`SnapshotsSubscriber` is used by [ExecutorPodsSnapshotsStoreImpl](ExecutorPodsSnapshotsStoreImpl.md) to manage [snapshot subscribers](ExecutorPodsSnapshotsStoreImpl.md#subscribers) that are notified about new [ExecutorPodsSnapshot](ExecutorPodsSnapshot.md)s regularly.

`SnapshotsSubscriber` manages [snapshotsBuffer](#snapshotsBuffer) queue of [ExecutorPodsSnapshot](ExecutorPodsSnapshot.md)s.

`SnapshotsSubscriber` is a `private class` of [ExecutorPodsSnapshotsStoreImpl](ExecutorPodsSnapshotsStoreImpl.md) with full access to its internals.

## Creating Instance

`SnapshotsSubscriber` takes the following to be created:

* [onNewSnapshots Callback Function](#onNewSnapshots)

`SnapshotsSubscriber` is created when:

* `ExecutorPodsSnapshotsStoreImpl` is requested to [add a subscriber](ExecutorPodsSnapshotsStoreImpl.md#addSubscriber)

## <span id="onNewSnapshots"> onNewSnapshots Callback Function

```scala
onNewSnapshots: Seq[ExecutorPodsSnapshot] => Unit
```

`SnapshotsSubscriber` is given a `onNewSnapshots` callback function when [created](#creating-instance).

## <span id="snapshotsBuffer"> snapshotsBuffer Queue

```scala
snapshotsBuffer: LinkedBlockingQueue[ExecutorPodsSnapshot]
```

`SnapshotsSubscriber` manages a `snapshotsBuffer` queue of [ExecutorPodsSnapshot](ExecutorPodsSnapshot.md)s.

## <span id="notificationCount"> notificationCount Counter

```scala
notificationCount: AtomicInteger
```

`SnapshotsSubscriber` manages a `notificationCount` counter.

## <span id="addCurrentSnapshot"> addCurrentSnapshot

```scala
addCurrentSnapshot(): Unit
```

`addCurrentSnapshot` requests the [snapshotsBuffer](#snapshotsBuffer) internal queue to add the [current ExecutorPodsSnapshot](ExecutorPodsSnapshotsStoreImpl.md#currentSnapshot).

`addCurrentSnapshot` is used when:

* `ExecutorPodsSnapshotsStoreImpl` is requested to [add a subscriber](ExecutorPodsSnapshotsStoreImpl.md#addSubscriber) and [addCurrentSnapshotToSubscribers](ExecutorPodsSnapshotsStoreImpl.md#addCurrentSnapshotToSubscribers)

## <span id="processSnapshots"> processSnapshots

```scala
processSnapshots(): Unit
```

`processSnapshots` increments the [notificationCount](#notificationCount) counter followed by [processSnapshotsInternal](#processSnapshotsInternal).

`processSnapshots` is used when:

* [ScheduledExecutorService](ExecutorPodsSnapshotsStoreImpl.md#subscribersExecutor) (of `ExecutorPodsSnapshotsStoreImpl`) executes regularly (for a subscriber)
* `ExecutorPodsSnapshotsStoreImpl` is requested to [notify all subscribers](ExecutorPodsSnapshotsStoreImpl.md#notifySubscribers)

### <span id="processSnapshotsInternal"> processSnapshotsInternal

```scala
processSnapshotsInternal(): Unit
```

`processSnapshotsInternal`...FIXME

!!! note
    `processSnapshotsInternal` can be called again recursively when the [notificationCount](#notificationCount) counter has been incremented while the `processSnapshotsInternal` was running.
