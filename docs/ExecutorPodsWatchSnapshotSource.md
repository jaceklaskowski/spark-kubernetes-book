# ExecutorPodsWatchSnapshotSource

`ExecutorPodsWatchSnapshotSource` is given [KubernetesClient](#kubernetesClient) to a Kubernetes API server to watch for status updates of the executor pods of a given Spark application when [started](#start) (that [ExecutorPodsWatcher](ExecutorPodsWatcher.md) passes along to the [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md#updatePod)).

## Creating Instance

`ExecutorPodsWatchSnapshotSource` takes the following to be created:

* <span id="snapshotsStore"> [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md)
* <span id="kubernetesClient"> `KubernetesClient`

`ExecutorPodsWatchSnapshotSource` is created when:

* `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)

## <span id="watchConnection"> watchConnection

```scala
watchConnection: Closeable
```

`ExecutorPodsWatchSnapshotSource` defines `watchConnection` internal registry to be a "watch connection" to a Kubernetes API server to watch any status updates of the executor pods of a given Spark application (using [ExecutorPodsWatcher](ExecutorPodsWatcher.md)).

`ExecutorPodsWatchSnapshotSource` uses `watchConnection` internal registry as an indication of whether it has been [started](#start) already or not (and throws an `IllegalArgumentException` when it has).

`ExecutorPodsWatchSnapshotSource` requests the `watchConnection` to close and `null`s it when requested to [stop](#stop).

## <span id="start"> Starting

```scala
start(
  applicationId: String): Unit
```

`start` prints out the following DEBUG message to the logs:

```text
Starting watch for pods with labels spark-app-selector=[applicationId], spark-role=executor.
```

`start` requests the [KubernetesClient](#kubernetesClient) to watch pods with the following labels and values and pass pod updates to [ExecutorPodsWatcher](ExecutorPodsWatcher.md).

Label Name | Value
-----------|----------
 `spark-app-selector` | the given `applicationId`
`spark-role` | `executor`

`start` is used when:

* `KubernetesClusterSchedulerBackend` is requested to [start](KubernetesClusterSchedulerBackend.md#start)

## Logging

Enable `ALL` logging level for `org.apache.spark.scheduler.cluster.k8s.ExecutorPodsWatchSnapshotSource` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```text
log4j.logger.org.apache.spark.scheduler.cluster.k8s.ExecutorPodsWatchSnapshotSource=ALL
```

Refer to [Logging](spark-logging.md).
