# PollRunnable

`PollRunnable` is a Java [Runnable]({{ java.api }}/java.base/java/lang/Runnable.html) that [ExecutorPodsPollingSnapshotSource](ExecutorPodsPollingSnapshotSource.md) schedules for [full executor pod state snapshots from Kubernetes](#run) every [spark.kubernetes.executor.apiPollingInterval](configuration-properties.md#spark.kubernetes.executor.apiPollingInterval) in the [Spark application](#applicationId).

`PollRunnable` is an internal class of [ExecutorPodsPollingSnapshotSource](ExecutorPodsPollingSnapshotSource.md) with full access to its internals.

## Creating Instance

`PollRunnable` takes the following to be created:

* <span id="applicationId"> Application Id

`PollRunnable` is created when:

* `ExecutorPodsPollingSnapshotSource` is requested to [start](ExecutorPodsPollingSnapshotSource.md#start)

## <span id="run"> Starting Thread

```scala
run(): Unit
```

`run` prints out the following DEBUG message to the logs:

```text
Resynchronizing full executor pod state from Kubernetes.
```

`run` requests the [KubernetesClient](ExecutorPodsPollingSnapshotSource.md#kubernetesClient) for Spark executor pods with the following labels and values.

Label Name | Value
-----------|----------
 `spark-app-selector` | [application Id](#applicationId)
`spark-role` | `executor`
`spark-exec-inactive` | `false` or not attached

In the end, `run` requests the [ExecutorPodsSnapshotsStore](ExecutorPodsPollingSnapshotSource.md#snapshotsStore) to [replace the snapshot](ExecutorPodsSnapshotsStore.md#replaceSnapshot).

## Logging

`PollRunnable` uses [org.apache.spark.scheduler.cluster.k8s.ExecutorPodsPollingSnapshotSource](ExecutorPodsPollingSnapshotSource.md#logging) logger for logging.
