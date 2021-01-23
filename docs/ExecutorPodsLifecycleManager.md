# ExecutorPodsLifecycleManager

## Creating Instance

`ExecutorPodsLifecycleManager` takes the following to be created:

* <span id="conf"> `SparkConf`
* <span id="kubernetesClient"> `KubernetesClient`
* <span id="snapshotsStore"> [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md)
* <span id="removedExecutorsCache"> Guava `Cache`

`ExecutorPodsLifecycleManager` is created when `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend) (and creates a [KubernetesClusterSchedulerBackend](KubernetesClusterSchedulerBackend.md#lifecycleEventHandler)).

## Configuration Properties

### <span id="eventProcessingInterval"> spark.kubernetes.executor.eventProcessingInterval

`ExecutorPodsLifecycleManager` uses the [spark.kubernetes.executor.eventProcessingInterval](configuration-properties.md#spark.kubernetes.executor.eventProcessingInterval) configuration property when [started](#start) to register a new subscriber for how often to...FIXME

### <span id="shouldDeleteExecutors"> spark.kubernetes.executor.deleteOnTermination

`ExecutorPodsLifecycleManager` uses the [spark.kubernetes.executor.deleteOnTermination](configuration-properties.md#spark.kubernetes.executor.deleteOnTermination) configuration property for [onFinalNonDeletedState](#onFinalNonDeletedState).

## <span id="missingPodDetectDelta"> Missing Pod Timeout

`ExecutorPodsLifecycleManager` defines **Missing Pod Timeout** based on the [spark.kubernetes.executor.missingPodDetectDelta](configuration-properties.md#spark.kubernetes.executor.missingPodDetectDelta) configuration property.

`ExecutorPodsLifecycleManager` uses the timeout to detect lost executor pods when [handling executor pods snapshots](#onNewSnapshots).

## <span id="start"> Starting

```scala
start(
  schedulerBackend: KubernetesClusterSchedulerBackend): Unit
```

`start` requests the [ExecutorPodsSnapshotsStore](#snapshotsStore) to [add a subscriber](ExecutorPodsSnapshotsStore.md#addSubscriber) to [intercept state changes in executor pods](#onNewSnapshots).

`start` is used when `KubernetesClusterSchedulerBackend` is [started](KubernetesClusterSchedulerBackend.md#start).

## <span id="onNewSnapshots"> Processing Executor Pods Snapshots

```scala
onNewSnapshots(
  schedulerBackend: KubernetesClusterSchedulerBackend,
  snapshots: Seq[ExecutorPodsSnapshot]): Unit
```

`onNewSnapshots` creates an empty `execIdsRemovedInThisRound` collection of executors to be removed.

`onNewSnapshots` walks over the input `ExecutorPodsSnapshot`s and branches off based on `ExecutorPodState`:

* For `PodDeleted`, `onNewSnapshots` prints out the following DEBUG message to the logs:

    ```text
    Snapshot reported deleted executor with id [execId], pod name [state.pod.getMetadata.getName]
    ```

    `onNewSnapshots` [removeExecutorFromSpark](#removeExecutorFromSpark) and adds the executor ID to the `execIdsRemovedInThisRound` local collection.

* For `PodFailed`, `onNewSnapshots` prints out the following DEBUG message to the logs:

    ```text
    Snapshot reported failed executor with id [execId], pod name [state.pod.getMetadata.getName]
    ```

    `onNewSnapshots` [onFinalNonDeletedState](#onFinalNonDeletedState) with the `execIdsRemovedInThisRound` local collection.

* For `PodSucceeded`, `onNewSnapshots` requests the input `KubernetesClusterSchedulerBackend` to [isExecutorActive](KubernetesClusterSchedulerBackend.md#isExecutorActive). If so, `onNewSnapshots` prints out the following INFO message to the logs:

    ```text
    Snapshot reported succeeded executor with id [execId], even though the application has not requested for it to be removed.
    ```

    Otherwise, `onNewSnapshots` prints out the following DEBUG message to the logs:

    ```text
    Snapshot reported succeeded executor with id [execId], pod name [state.pod.getMetadata.getName].
    ```

    `onNewSnapshots` [onFinalNonDeletedState](#onFinalNonDeletedState) with the `execIdsRemovedInThisRound` local collection.

### <span id="onFinalNonDeletedState"> onFinalNonDeletedState

```scala
onFinalNonDeletedState(
  podState: FinalPodState,
  execId: Long,
  schedulerBackend: KubernetesClusterSchedulerBackend,
  deleteFromK8s: Boolean): Boolean
```

`onFinalNonDeletedState` [removeExecutorFromSpark](#removeExecutorFromSpark) (and records the `deleted` return flag to be returned in the end).

With the given `deleteFromK8s` flag enabled, `onFinalNonDeletedState` [removeExecutorFromK8s](#removeExecutorFromK8s).

### <span id="removeExecutorFromSpark"> removeExecutorFromSpark

```scala
removeExecutorFromSpark(
  schedulerBackend: KubernetesClusterSchedulerBackend,
  podState: FinalPodState,
  execId: Long): Unit
```

`removeExecutorFromSpark`...FIXME

### <span id="removeExecutorFromK8s"> removeExecutorFromK8s

```scala
removeExecutorFromK8s(
  execId: Long,
  updatedPod: Pod): Unit
```

`removeExecutorFromK8s`...FIXME

## Logging

Enable `ALL` logging level for `org.apache.spark.scheduler.cluster.k8s.ExecutorPodsLifecycleManager` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```text
log4j.logger.org.apache.spark.scheduler.cluster.k8s.ExecutorPodsLifecycleManager=ALL
```

Refer to [Logging](spark-logging.md).