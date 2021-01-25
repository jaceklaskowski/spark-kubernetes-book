# ExecutorPodsPollingSnapshotSource

`ExecutorPodsPollingSnapshotSource` manages [pollingFuture](#pollingFuture) (for [PollRunnable](PollRunnable.md)s) to synchronize executor pods state every [spark.kubernetes.executor.apiPollingInterval](#pollingInterval) (by requesting the [ExecutorPodsSnapshotsStore](#snapshotsStore) to [replace executor pods snapshot](ExecutorPodsSnapshotsStore.md#replaceSnapshot)).

## Creating Instance

`ExecutorPodsPollingSnapshotSource` takes the following to be created:

* <span id="conf"> `SparkConf`
* <span id="kubernetesClient"> `KubernetesClient`
* <span id="snapshotsStore"> [ExecutorPodsSnapshotsStore](ExecutorPodsSnapshotsStore.md)
* <span id="pollingExecutor"> Java [ScheduledExecutorService]({{ java.api }}/java.base/java/util/concurrent/ScheduledExecutorService.html)

`ExecutorPodsPollingSnapshotSource` is created when:

* `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend) (and creates a [KubernetesClusterSchedulerBackend](KubernetesClusterSchedulerBackend.md#pollEvents))

![ExecutorPodsPollingSnapshotSource](images/ExecutorPodsPollingSnapshotSource.png)

## <span id="pollingInterval"> spark.kubernetes.executor.apiPollingInterval

`ExecutorPodsPollingSnapshotSource` uses [spark.kubernetes.executor.apiPollingInterval](configuration-properties.md#spark.kubernetes.executor.apiPollingInterval) configuration property when [started](#start) (to schedule a [PollRunnable](PollRunnable.md) for regular executor pod state synchronization).

## <span id="pollingFuture"> pollingFuture

```scala
pollingFuture: Future[_]
```

`pollingFuture`...FIXME

## <span id="start"> Starting

```scala
start(
  applicationId: String): Unit
```

![ExecutorPodsPollingSnapshotSource and KubernetesClusterSchedulerBackend](images/ExecutorPodsPollingSnapshotSource-start.png)

`start` prints out the following DEBUG message to the logs (with the [pollingInterval](#pollingInterval)):

```text
Starting to check for executor pod state every [pollingInterval] ms.
```

`start` throws an `IllegalArgumentException` when started twice (i.e. [pollingFuture](#pollingFuture) has already been initialized):

```text
Cannot start polling more than once.
```

`start` is used when:

* `KubernetesClusterSchedulerBackend` is requested to [start](KubernetesClusterSchedulerBackend.md#start)

## Logging

Enable `ALL` logging level for `org.apache.spark.scheduler.cluster.k8s.ExecutorPodsPollingSnapshotSource` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```text
log4j.logger.org.apache.spark.scheduler.cluster.k8s.ExecutorPodsPollingSnapshotSource=ALL
```

Refer to [Logging](spark-logging.md).
