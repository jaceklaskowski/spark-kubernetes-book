# ExecutorPodsWatcher

`ExecutorPodsWatcher` is a Kubernetes' `Watcher[Pod]` to [be notified about executor pod updates](#eventReceived) (and [update](ExecutorPodsSnapshotsStore.md#updatePod) the [ExecutorPodsSnapshotsStore](ExecutorPodsWatchSnapshotSource.md#snapshotsStore) of the parent [ExecutorPodsWatchSnapshotSource](ExecutorPodsWatchSnapshotSource.md)).

`ExecutorPodsWatcher` is an internal class of [ExecutorPodsWatchSnapshotSource](ExecutorPodsWatchSnapshotSource.md) with full access to its internals.

## Creating Instance

`ExecutorPodsWatcher` takes no arguments to be created.

`ExecutorPodsWatcher` is created when:

* `ExecutorPodsWatchSnapshotSource` is requested to [start](ExecutorPodsWatchSnapshotSource.md#start)

## <span id="eventReceived"> Executor Pod Update

```scala
eventReceived(
  action: Action,
  pod: Pod): Unit
```

`eventReceived` is part of the Kubernetes Client `Watcher` abstraction.

`eventReceived` prints out the following DEBUG message to the logs:

```text
Received executor pod update for pod named [podName], action [action]
```

`eventReceived` requests the [ExecutorPodsSnapshotsStore](ExecutorPodsWatchSnapshotSource.md#snapshotsStore) (of the parent [ExecutorPodsWatchSnapshotSource](ExecutorPodsWatchSnapshotSource.md)) to [updatePod](ExecutorPodsSnapshotsStore.md#updatePod).

## <span id="onClose"> Close Notification

```scala
onClose(
  e: KubernetesClientException): Unit
```

`onClose` is part of the Kubernetes Client `Watcher` abstraction.

`onClose` prints out the following WARN message to the logs:

```text
Kubernetes client has been closed (this is expected if the application is shutting down.)
```

## Logging

`ExecutorPodsWatcher` uses [org.apache.spark.scheduler.cluster.k8s.ExecutorPodsWatchSnapshotSource](ExecutorPodsWatchSnapshotSource.md#logging) logger for logging.
