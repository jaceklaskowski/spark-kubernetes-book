# ExecutorPodsSnapshot

`ExecutorPodsSnapshot` is an immutable view (_snapshot_) of the executor pods running in a Kubernetes cluster.

`ExecutorPodsSnapshot` tracks the [state of executor pods](#executorPods) and can be updated [partially](#withUpdate) or [fully](#fullSnapshotTs).

## Creating Instance

`ExecutorPodsSnapshot` takes the following to be created:

* <span id="executorPods"> `ExecutorPodState` by Executor ID (`Map[Long, ExecutorPodState]`)
* [Full Snapshot Timestamp](#fullSnapshotTs)

`ExecutorPodsSnapshot` is created (indirectly using [apply](#apply) utility) when:

* `ExecutorPodsAllocator` is [created](ExecutorPodsAllocator.md#lastSnapshot)
* `ExecutorPodsSnapshotsStoreImpl` is [created](ExecutorPodsSnapshotsStoreImpl.md#currentSnapshot) and requested to [replace a snapshot](ExecutorPodsSnapshotsStoreImpl.md#replaceSnapshot)
* `ExecutorPodsSnapshot` is requested to [update](#withUpdate)

## <span id="fullSnapshotTs"> Full Snapshot Timestamp

`ExecutorPodsSnapshot` keeps track of the time of the full snapshot of executor pods.

The time is given (via [apply](#apply)) when:

* `ExecutorPodsSnapshotsStoreImpl` is requested to [replace a snapshot](ExecutorPodsSnapshotsStoreImpl.md#replaceSnapshot) (and basically do a full synchronization)

## <span id="shouldCheckAllContainers"><span id="setShouldCheckAllContainers"> shouldCheckAllContainers Flag

```scala
shouldCheckAllContainers: Boolean
```

`ExecutorPodsSnapshot` uses `shouldCheckAllContainers` internal flag to control whether or not to include all containers in a running pod when requested for the [ExecutorPodState](#toState).

`shouldCheckAllContainers` is controlled by [spark.kubernetes.executor.checkAllContainers](configuration-properties.md#spark.kubernetes.executor.checkAllContainers) configuration property (when `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)).

## <span id="apply"> apply Utility

```scala
apply(): ExecutorPodsSnapshot
apply(
  executorPods: Seq[Pod],
  fullSnapshotTs: Long): ExecutorPodsSnapshot
```

`apply` creates a [ExecutorPodsSnapshot](#creating-instance) with the given arguments.

`apply` is used when:

* `ExecutorPodsAllocator` is [created](ExecutorPodsAllocator.md#lastSnapshot)
* `ExecutorPodsSnapshotsStoreImpl` is [created](ExecutorPodsSnapshotsStoreImpl.md#currentSnapshot) and requested to [replace a snapshot](ExecutorPodsSnapshotsStoreImpl.md#replaceSnapshot)

## <span id="withUpdate"> Updating ExecutorPodsSnapshot

```scala
withUpdate(
  updatedPod: Pod): ExecutorPodsSnapshot
```

`withUpdate` creates a new [ExecutorPodsSnapshot](#creating-instance) with the [executorPods](#executorPods) updated based on the given executor pod update ([converted](#toStatesByExecutorId)).

`withUpdate` is used when:

* `ExecutorPodsSnapshotsStoreImpl` is requested to [updatePod](ExecutorPodsSnapshotsStoreImpl.md#updatePod)

## <span id="toStatesByExecutorId"> toStatesByExecutorId

```scala
toStatesByExecutorId(
  executorPods: Seq[Pod]): Map[Long, ExecutorPodState]
```

`toStatesByExecutorId`...FIXME

`toStatesByExecutorId` is used when:

* `ExecutorPodsSnapshot` is requested to [withUpdate](#withUpdate) and [apply](#apply)

### <span id="toState"> toState

```scala
toState(
  pod: Pod): ExecutorPodState
```

`toState`...FIXME

### <span id="isDeleted"> isDeleted

```scala
isDeleted(
  pod: Pod): Boolean
```

`isDeleted`...FIXME
