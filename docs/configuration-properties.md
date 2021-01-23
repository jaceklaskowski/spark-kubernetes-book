# Configuration Properties

## <span id="spark.kubernetes.allocation.batch.delay"><span id="KUBERNETES_ALLOCATION_BATCH_DELAY"> spark.kubernetes.allocation.batch.delay

Time (in millis) to wait between each round of executor allocation

Default: `1s`

Used when:

* `ExecutorPodsAllocator` is [created](ExecutorPodsAllocator.md#podAllocationDelay)

## <span id="spark.kubernetes.allocation.batch.size"><span id="KUBERNETES_ALLOCATION_BATCH_SIZE"> spark.kubernetes.allocation.batch.size

Minimum number of executor pods to allocate at once in each round of executor allocation

Default: `5`

Used when:

* `ExecutorPodsAllocator` is [created](ExecutorPodsAllocator.md#podAllocationSize)

## <span id="spark.kubernetes.allocation.executor.timeout"><span id="KUBERNETES_ALLOCATION_EXECUTOR_TIMEOUT"> spark.kubernetes.allocation.executor.timeout

Time (in millis) to wait before a pending executor is considered timed out

Default: `600s`

Used when:

* `ExecutorPodsAllocator` is requested to [handle executor pods snapshots](ExecutorPodsAllocator.md#podCreationTimeout)

## <span id="spark.kubernetes.authenticate"><span id="KUBERNETES_AUTH_CLIENT_MODE_PREFIX"> spark.kubernetes.authenticate

FIXME

## <span id="spark.kubernetes.authenticate.driver.mounted"><span id="KUBERNETES_AUTH_DRIVER_MOUNTED_CONF_PREFIX"> spark.kubernetes.authenticate.driver.mounted

FIXME

## <span id="spark.kubernetes.authenticate.executor.serviceAccountName"><span id="KUBERNETES_EXECUTOR_SERVICE_ACCOUNT_NAME"> spark.kubernetes.authenticate.executor.serviceAccountName

Service account for executor pods

Default: (undefined)

Used when:

* `ExecutorKubernetesCredentialsFeatureStep` is requested to [configure a pod](ExecutorKubernetesCredentialsFeatureStep.md#executorServiceAccount)

## <span id="spark.kubernetes.configMap.maxSize"><span id="CONFIG_MAP_MAXSIZE"> spark.kubernetes.configMap.maxSize

Max size limit (`long`) for a config map. Configurable as per https://etcd.io/docs/v3.4.0/dev-guide/limit/ on k8s server end.

Default: `1572864` (`1.5 MiB`)

Used when:

* `KubernetesClientUtils` utility is used to [loadSparkConfDirFiles](KubernetesClientUtils.md#loadSparkConfDirFiles)

## <span id="spark.kubernetes.container.image"><span id="CONTAINER_IMAGE"> spark.kubernetes.container.image

Container image to use for Spark containers (unless [spark.kubernetes.driver.container.image](#spark.kubernetes.driver.container.image) or [spark.kubernetes.executor.container.image](#spark.kubernetes.executor.container.image) are defined)

Default: (undefined)

## <span id="spark.kubernetes.container.image.pullPolicy"><span id="CONTAINER_IMAGE_PULL_POLICY"> spark.kubernetes.container.image.pullPolicy

Kubernetes image pull policy:

* `Always`
* `Never`
* `IfNotPresent`

Default: `IfNotPresent`

Used when:

* `KubernetesConf` is requested for [imagePullPolicy](KubernetesConf.md#imagePullPolicy)

## <span id="spark.kubernetes.context"><span id="KUBERNETES_CONTEXT"> spark.kubernetes.context

The desired context from your K8S config file used to configure the K8S client for interacting with the cluster. Useful if your config file has multiple clusters or user identities defined. The client library used locates the config file via the `KUBECONFIG` environment variable or by defaulting to `.kube/config` under your home directory. If not specified then your current context is used.  You can always override specific aspects of the config file provided configuration using other Spark on K8S configuration options.

Default: (undefined)

Used when:

* `SparkKubernetesClientFactory` is requested to [create a KubernetesClient](SparkKubernetesClientFactory.md#createKubernetesClient)

## <span id="spark.kubernetes.driver.container.image"><span id="DRIVER_CONTAINER_IMAGE"> spark.kubernetes.driver.container.image

Container image for drivers

Default: [spark.kubernetes.container.image](#spark.kubernetes.container.image)

Used when:

* `BasicDriverFeatureStep` is requested for a [driverContainerImage](BasicDriverFeatureStep.md#driverContainerImage)

## <span id="spark.kubernetes.driver.master"><span id="KUBERNETES_DRIVER_MASTER_URL"> spark.kubernetes.driver.master

The internal Kubernetes master (API server) address to be used for driver to request executors.

Default: `https://kubernetes.default.svc`

## <span id="spark.kubernetes.driver.pod.name"><span id="KUBERNETES_DRIVER_POD_NAME"> spark.kubernetes.driver.pod.name

Name of the driver pod

Default: (undefined)

Must be provided if a Spark application is deployed in [cluster deploy mode](KubernetesClusterManager.md#createSchedulerBackend)

Used when:

* `BasicDriverFeatureStep` is requested for the [driverPodName](BasicDriverFeatureStep.md#driverPodName) (and [additional system properties of a driver pod](BasicDriverFeatureStep.md#getAdditionalPodSystemProperties))
* `ExecutorPodsAllocator` is requested for the [kubernetesDriverPodName](ExecutorPodsAllocator.md#kubernetesDriverPodName)

## <span id="spark.kubernetes.driver.podTemplateContainerName"><span id="KUBERNETES_DRIVER_PODTEMPLATE_CONTAINER_NAME"> spark.kubernetes.driver.podTemplateContainerName

Name of the driver container in a [pod template](#spark.kubernetes.driver.podTemplateFile)

Default: (undefined)

Used when:

* `KubernetesDriverBuilder` is requested for a [driver pod specification](KubernetesDriverBuilder.md#buildFromFeatures)

## <span id="spark.kubernetes.driver.podTemplateFile"><span id="KUBERNETES_DRIVER_PODTEMPLATE_FILE"> spark.kubernetes.driver.podTemplateFile

Pod template file for drivers (in [cluster](overview.md#cluster-deploy-mode) deploy mode)

Default: (undefined)

Used when:

* `KubernetesDriverBuilder` is requested for a [driver pod specification](KubernetesDriverBuilder.md#buildFromFeatures)

## <span id="spark.kubernetes.driver.request.cores"><span id="KUBERNETES_DRIVER_REQUEST_CORES"> spark.kubernetes.driver.request.cores

Specify the `cpu` request for the driver pod

Default: (undefined)

Used when:

* `BasicDriverFeatureStep` is requested to [configure a pod](BasicDriverFeatureStep.md#configurePod)

## <span id="spark.kubernetes.executor.apiPollingInterval"><span id="KUBERNETES_EXECUTOR_API_POLLING_INTERVAL"> spark.kubernetes.executor.apiPollingInterval

Interval (in millis) between polls against the Kubernetes API server to inspect the state of executors.

Default: `30s`

Used when:

* `ExecutorPodsPollingSnapshotSource` is requested to [start](ExecutorPodsPollingSnapshotSource.md#pollingInterval)

## <span id="spark.kubernetes.executor.checkAllContainers"><span id="KUBERNETES_EXECUTOR_CHECK_ALL_CONTAINERS"> spark.kubernetes.executor.checkAllContainers

Controls whether or not to check the status of all containers in a running executor pod when [reporting executor status](ExecutorPodsSnapshot.md#shouldCheckAllContainers)

Default: `false`

Used when:

* `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)

## <span id="spark.kubernetes.executor.container.image"><span id="EXECUTOR_CONTAINER_IMAGE"> spark.kubernetes.executor.container.image

Container image for executors

Default: [spark.kubernetes.container.image](#spark.kubernetes.container.image)

Used when:

* `BasicExecutorFeatureStep` is requested for a [driverContainerImage](BasicExecutorFeatureStep.md#executorContainerImage)

## <span id="spark.kubernetes.executor.deleteOnTermination"><span id="KUBERNETES_DELETE_EXECUTORS"> spark.kubernetes.executor.deleteOnTermination

Controls whether or not to delete executor pods after they have finished (successfully or not)

Default: `true`

Used when:

* `ExecutorPodsAllocator` is requested to [handle executor pods snapshots](ExecutorPodsAllocator.md#onNewSnapshots)
* `ExecutorPodsLifecycleManager` is requested to [handle executor pods snapshots](ExecutorPodsLifecycleManager.md#onFinalNonDeletedState)
* `KubernetesClusterSchedulerBackend` is requested to [stop](KubernetesClusterSchedulerBackend.md#stop)

## <span id="spark.kubernetes.executor.eventProcessingInterval"><span id="KUBERNETES_EXECUTOR_EVENT_PROCESSING_INTERVAL"> spark.kubernetes.executor.eventProcessingInterval

Interval (in millis) between successive inspection of executor events sent from the Kubernetes API

Default: `1s`

Used when:

* `ExecutorPodsLifecycleManager` is requested to [start and register a new subscriber](ExecutorPodsLifecycleManager.md#eventProcessingInterval)

## <span id="spark.kubernetes.executor.missingPodDetectDelta"><span id="KUBERNETES_EXECUTOR_MISSING_POD_DETECT_DELTA"> spark.kubernetes.executor.missingPodDetectDelta

Time (in millis) to wait before an executor is removed due to the executor's pod being missed in the Kubernetes API server's polled list of pods

Default: `30s`

Used when:

* `ExecutorPodsLifecycleManager` is requested to [handle executor pods snapshots](ExecutorPodsLifecycleManager.md#missingPodDetectDelta)

## <span id="spark.kubernetes.executor.podNamePrefix"><span id="KUBERNETES_EXECUTOR_POD_NAME_PREFIX"> spark.kubernetes.executor.podNamePrefix

**(internal)** Prefix of the executor pod names

Default: (undefined)

Unless defined, it is set explicitly when `KubernetesClusterManager` is requested to [create a SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)

Used when:

* `KubernetesExecutorConf` is requested for the [resourceNamePrefix](KubernetesExecutorConf.md#resourceNamePrefix)

## <span id="spark.kubernetes.executor.podTemplateContainerName"><span id="KUBERNETES_EXECUTOR_PODTEMPLATE_CONTAINER_NAME"> spark.kubernetes.executor.podTemplateContainerName

Name of the container for executors in a pod template

Default: (undefined)

Used when:

* `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)
* `KubernetesExecutorBuilder` is requested for a [pod spec for executors](KubernetesExecutorBuilder.md#buildFromFeatures)

## <span id="spark.kubernetes.executor.podTemplateFile"><span id="KUBERNETES_EXECUTOR_PODTEMPLATE_FILE"> spark.kubernetes.executor.podTemplateFile

Pod template file for executors

Default: (undefined)

Used when:

* `KubernetesClusterManager` is requested to [create a SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)
* `KubernetesExecutorBuilder` is requested to [build an executor pod](KubernetesExecutorBuilder.md#buildFromFeatures)
* `PodTemplateConfigMapStep` is [created](PodTemplateConfigMapStep.md#hasTemplate) and requested to [configurePod](PodTemplateConfigMapStep.md#configurePod), [getAdditionalPodSystemProperties](PodTemplateConfigMapStep.md#getAdditionalPodSystemProperties), [getAdditionalKubernetesResources](PodTemplateConfigMapStep.md#getAdditionalKubernetesResources)

## <span id="spark.kubernetes.executor.request.cores"><span id="KUBERNETES_EXECUTOR_REQUEST_CORES"> spark.kubernetes.executor.request.cores

Specifies the cpu quantity request for executor pods (to be more Kubernetes-oriented when requesting resources for executor pods than Spark scheduler's approach based on `spark.executor.cores`).

Default: (undefined)

Used when:

* `BasicExecutorFeatureStep` is requested to [configure an executor pod](BasicExecutorFeatureStep.md#configurePod)

## <span id="spark.kubernetes.executor.scheduler.name"><span id="KUBERNETES_EXECUTOR_SCHEDULER_NAME"> spark.kubernetes.executor.scheduler.name

Name of the scheduler for executor pods (a pod's `spec.schedulerName`)

Default: (undefined)

Used when:

* `BasicExecutorFeatureStep` is requested to [configure an executor pod](BasicExecutorFeatureStep.md#configurePod)

## <span id="spark.kubernetes.file.upload.path"><span id="KUBERNETES_FILE_UPLOAD_PATH"> spark.kubernetes.file.upload.path

Hadoop DFS-compatible file system path where files from the local file system will be uploded to in `cluster` deploy mode. The subdirectories (one per Spark application) with the local files are of the format `spark-upload-[uuid]`.

Default: (undefined)

Used when:

* `KubernetesUtils` is requested to [uploadFileUri](KubernetesUtils.md#uploadFileUri)

## <span id="spark.kubernetes.local.dirs.tmpfs"><span id="KUBERNETES_LOCAL_DIRS_TMPFS"> spark.kubernetes.local.dirs.tmpfs

If `true`, `emptyDir` volumes created to back `SPARK_LOCAL_DIRS` will have their medium set to `Memory` so that they will be created as tmpfs (i.e. RAM) backed volumes. This may improve performance but scratch space usage will count towards your pods memory limit so you may wish to request more memory.

Default: `false`

Used when:

* `LocalDirsFeatureStep` is requested to [configure a pod](LocalDirsFeatureStep.md#configurePod)

## <span id="spark.kubernetes.memoryOverheadFactor"><span id="MEMORY_OVERHEAD_FACTOR"> spark.kubernetes.memoryOverheadFactor

**Memory Overhead Factor** that will allocate memory to non-JVM jobs which in the case of JVM tasks will default to 0.10 and 0.40 for non-JVM jobs

Must be a double between (0, 1.0)

Default: `0.1`

Used when:

* `BasicDriverFeatureStep` is requested to [configure a pod](BasicDriverFeatureStep.md#configurePod)
* `BasicExecutorFeatureStep` is requested to [configure a pod](BasicExecutorFeatureStep.md#configurePod)

## <span id="spark.kubernetes.namespace"><span id="KUBERNETES_NAMESPACE"> spark.kubernetes.namespace

[Namespace]({{ k8s.doc }}/concepts/overview/working-with-objects/namespaces/) for running the driver and executor pods

Default: `default`

Used when:

* `KubernetesConf` is requested for [namespace](KubernetesConf.md#namespace)
* `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)
* `ExecutorPodsAllocator` is created (and initializes [namespace](ExecutorPodsAllocator.md#namespace))

## <span id="spark.kubernetes.report.interval"><span id="REPORT_INTERVAL"> spark.kubernetes.report.interval

Interval between reports of the current app status in cluster mode

Default: `1s`

Used when:

* `LoggingPodStatusWatcherImpl` is requested to [watchOrStop](LoggingPodStatusWatcherImpl.md#watchOrStop)

## <span id="spark.kubernetes.submission.waitAppCompletion"><span id="WAIT_FOR_APP_COMPLETION"> spark.kubernetes.submission.waitAppCompletion

In `cluster` deploy mode, whether to wait for the application to finish before exiting the launcher process.

Default: `true`

Used when:

* `LoggingPodStatusWatcherImpl` is requested to [watchOrStop](LoggingPodStatusWatcherImpl.md#watchOrStop)

## <span id="spark.kubernetes.submitInDriver"><span id="KUBERNETES_DRIVER_SUBMIT_CHECK"> spark.kubernetes.submitInDriver

**(internal)** Whether executing in `cluster` deploy mode

Default: `false`

`spark.kubernetes.submitInDriver` is `true` in [BasicDriverFeatureStep](BasicDriverFeatureStep.md#getAdditionalPodSystemProperties).

Used when:

* `BasicDriverFeatureStep` is requested to [getAdditionalPodSystemProperties](BasicDriverFeatureStep.md#getAdditionalPodSystemProperties)
* `KubernetesClusterManager` is requested for a [SchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)
