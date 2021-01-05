# KubernetesClusterManager

<span id="canCreate">
`KubernetesClusterManager` is an `ExternalClusterManager` ([Apache Spark]({{ book.spark_core }}/scheduler/ExternalClusterManager)) that can create scheduler components for **k8s** master URLs.

`KubernetesClusterManager` is registered with Apache Spark using `META-INF/services/org.apache.spark.scheduler.ExternalClusterManager` service file.

## Creating Instance

`KubernetesClusterManager` takes no arguments to be created.

`KubernetesClusterManager` is created when:

* `SparkContext` is requested for an `ExternalClusterManager` (when requested for a [SchedulerBackend and TaskScheduler]({{ book.spark_core }}/SparkContext/#getClusterManager))

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

`createSchedulerBackend` determines four internal values based on the [spark.kubernetes.submitInDriver](configuration-properties.md#spark.kubernetes.submitInDriver) internal configuration property.

&nbsp; | spark.kubernetes.submitInDriver | &nbsp;
-------|---------------------------------|-
&nbsp;                         | Enabled (`true`) | Disabled (`false`)
<span id="createSchedulerBackend-authConfPrefix"> **authConfPrefix**             | `spark.kubernetes.authenticate.driver.mounted` | `spark.kubernetes.authenticate`
<span id="createSchedulerBackend-apiServerUri"> **apiServerUri**               | [spark.kubernetes.driver.master](configuration-properties.md#spark.kubernetes.driver.master) | Master URL with no **k8s://** prefix
<span id="createSchedulerBackend-defaultServiceAccountToken"> **defaultServiceAccountToken** | `/var/run/secrets/kubernetes.io/serviceaccount/token`  | &nbsp;
<span id="createSchedulerBackend-defaultServiceAccountCaCrt"> **defaultServiceAccountCaCrt** | `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt` | &nbsp;

Unless already defined, `createSchedulerBackend` sets the [spark.kubernetes.executor.podNamePrefix](configuration-properties.md#spark.kubernetes.executor.podNamePrefix) configuration properties based on [spark.app.name](KubernetesConf.md#getResourceNamePrefix) prefix.

`createSchedulerBackend` [creates a KubernetesClient](SparkKubernetesClientFactory.md#createKubernetesClient) for the `Driver` client type and the following:

* [spark.kubernetes.namespace](configuration-properties.md#spark.kubernetes.namespace) configuration property
* [apiServerUri](#createSchedulerBackend-apiServerUri)
* [authConfPrefix](#createSchedulerBackend-authConfPrefix)
* [defaultServiceAccountToken](#createSchedulerBackend-defaultServiceAccountToken)
* [defaultServiceAccountCaCrt](#createSchedulerBackend-defaultServiceAccountCaCrt)

With [spark.kubernetes.executor.podTemplateFile](configuration-properties.md#spark.kubernetes.executor.podTemplateFile) configuration property enabled, `createSchedulerBackend` [loads the pod spec](KubernetesUtils.md#loadPodFromTemplate) from the pod template file with the optional [spark.kubernetes.executor.podTemplateContainerName](configuration-properties.md#spark.kubernetes.executor.podTemplateContainerName) configuration property.

In the end, `createSchedulerBackend` creates a [KubernetesClusterSchedulerBackend](KubernetesClusterSchedulerBackend.md) with the following:

* Java `ScheduledExecutorService` with **kubernetes-executor-maintenance** thread name

* [ExecutorPodsSnapshotsStoreImpl](ExecutorPodsSnapshotsStoreImpl.md) with a Java `ScheduledExecutorService` with **kubernetes-executor-snapshots-subscribers** thread names and 2 threads

* [ExecutorPodsLifecycleManager](ExecutorPodsLifecycleManager.md)

* [ExecutorPodsAllocator](ExecutorPodsAllocator.md)

* [ExecutorPodsWatchSnapshotSource](ExecutorPodsWatchSnapshotSource.md)

* [ExecutorPodsPollingSnapshotSource](ExecutorPodsPollingSnapshotSource.md) with a Java `ScheduledExecutorService` with **kubernetes-executor-pod-polling-sync** thread name

### IllegalArgumentException

With `spark.kubernetes.submitInDriver` enabled, `createSchedulerBackend` asserts that the name of the driver pod is configured (using [spark.kubernetes.driver.pod.name](configuration-properties.md#spark.kubernetes.driver.pod.name) configuration property) or else throws an `IllegalArgumentException`:

```text
If the application is deployed using spark-submit in cluster mode, the driver pod name must be provided.
```

## <span id="createTaskScheduler"> Creating TaskScheduler

```scala
createTaskScheduler(
  sc: SparkContext,
  masterURL: String): TaskScheduler
```

`createTaskScheduler` is part of the `ExternalClusterManager` ([Apache Spark]({{ book.spark_core }}/scheduler/ExternalClusterManager#createTaskScheduler)) abstraction.

`createTaskScheduler` creates a `TaskSchedulerImpl` ([Apache Spark]({{ book.spark_core }}/scheduler/TaskSchedulerImpl)).

## <span id="initialize"> Initializing Scheduling Components

```scala
initialize(
  scheduler: TaskScheduler,
  backend: SchedulerBackend): Unit
```

`initialize` is part of the `ExternalClusterManager` ([Apache Spark]({{ book.spark_core }}/scheduler/ExternalClusterManager#initialize)) abstraction.

`initialize` requests the given `TaskSchedulerImpl` ([Apache Spark]({{ book.spark_core }}/scheduler/TaskSchedulerImpl)) to initialize with the given `SchedulerBackend` ([Apache Spark]({{ book.spark_core }}/scheduler/SchedulerBackend/)).
