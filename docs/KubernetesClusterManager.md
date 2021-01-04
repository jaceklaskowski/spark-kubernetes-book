# KubernetesClusterManager

<span id="canCreate">
`KubernetesClusterManager` is an `ExternalClusterManager` ([Apache Spark]({{ book.spark_core }}/scheduler/ExternalClusterManager)) that supports **k8s** master URLs.

## Creating Instance

`KubernetesClusterManager` takes no arguments to be created.

`KubernetesClusterManager` is created when:

* `SparkContext` is requested for an `ExternalClusterManager` (for a certain master URL)

## <span id="createTaskScheduler"> Creating TaskScheduler

```scala
createTaskScheduler(
  sc: SparkContext,
  masterURL: String): TaskScheduler
```

`createTaskScheduler` is part of the `ExternalClusterManager` ([Apache Spark]({{ book.spark_core }}/scheduler/ExternalClusterManager#createTaskScheduler)) abstraction.

`createTaskScheduler` creates a `TaskSchedulerImpl`.

## <span id="createSchedulerBackend"> Creating SchedulerBackend

```scala
createSchedulerBackend(
  sc: SparkContext,
  masterURL: String,
  scheduler: TaskScheduler): SchedulerBackend
```

`createSchedulerBackend` is part of the `ExternalClusterManager` ([Apache Spark]({{ book.spark_core }}/scheduler/ExternalClusterManager#createSchedulerBackend)) abstraction.

`createSchedulerBackend` creates a [KubernetesClusterSchedulerBackend](KubernetesClusterSchedulerBackend.md).

!!! note
    `createSchedulerBackend` assumes that the given `TaskScheduler` is `TaskSchedulerImpl` ([Apache Spark]({{ book.spark_core }}/scheduler/TaskSchedulerImpl/)).

`createSchedulerBackend` uses [spark.kubernetes.submitInDriver](configuration-properties.md#spark.kubernetes.submitInDriver) configuration property to determine whether executing in `cluster` deploy mode.

`createSchedulerBackend`...FIXME

## <span id="initialize"> Initializing Scheduling Components

```scala
initialize(
  scheduler: TaskScheduler,
  backend: SchedulerBackend): Unit
```

`initialize` is part of the `ExternalClusterManager` ([Apache Spark]({{ book.spark_core }}/scheduler/ExternalClusterManager#initialize)) abstraction.

`initialize` requests the given `TaskSchedulerImpl` to [initialize](#initialize) with the given `SchedulerBackend`.
