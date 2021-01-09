# KubernetesUtils

## <span id="loadPodFromTemplate"> Loading Pod from Template File

```scala
loadPodFromTemplate(
  kubernetesClient: KubernetesClient,
  templateFile: File,
  containerName: Option[String]): SparkPod
```

`loadPodFromTemplate` requests the given `KubernetesClient` to load a pod for the input template file.

`loadPodFromTemplate` [selectSparkContainer](#selectSparkContainer) (with the pod and the input container name).

In case of an `Exception`, `loadPodFromTemplate` prints out the following ERROR message to the logs:

```text
Encountered exception while attempting to load initial pod spec from file
```

`loadPodFromTemplate` (re)throws a `SparkException`:

```text
Could not load pod from template file.
```

`loadPodFromTemplate` is used when:

* `KubernetesClusterManager` is requested to [createSchedulerBackend](KubernetesClusterManager.md#createSchedulerBackend)
* `KubernetesDriverBuilder` is requested to [buildFromFeatures](KubernetesDriverBuilder.md#buildFromFeatures)
* `KubernetesExecutorBuilder` is requested to [buildFromFeatures](KubernetesExecutorBuilder.md#buildFromFeatures)

## <span id="uploadAndTransformFileUris"> Uploading Local Files to Hadoop DFS

```scala
uploadAndTransformFileUris(
  fileUris: Iterable[String],
  conf: Option[SparkConf] = None): Iterable[String]
```

`uploadAndTransformFileUris` [uploads local files](#uploadFileUri) in the given `fileUris` to Hadoop DFS (based on [spark.kubernetes.file.upload.path](configuration-properties.md#spark.kubernetes.file.upload.path) configuration property).

In the end, `uploadAndTransformFileUris` returns the target URIs.

`uploadAndTransformFileUris` is used when:

* `BasicDriverFeatureStep` is requested to [getAdditionalPodSystemProperties](BasicDriverFeatureStep.md#getAdditionalPodSystemProperties)

### <span id="uploadFileUri"> uploadFileUri

```scala
uploadFileUri(
  uri: String,
  conf: Option[SparkConf] = None): String
```

`uploadFileUri` resolves the given `uri` to a well-formed `file` URI.

`uploadFileUri` creates a new Hadoop `Configuration` and resolves the [spark.kubernetes.file.upload.path](configuration-properties.md#spark.kubernetes.file.upload.path) configuration property to a Hadoop `FileSystem`.

`uploadFileUri` creates (_mkdirs_) the Hadoop DFS path to upload the file of the format:

```text
[spark.kubernetes.file.upload.path]/[spark-upload-[randomUUID]]
```

`uploadFileUri` prints out the following INFO message to the logs:

```text
Uploading file: [path] to dest: [targetUri]...
```

In the end, `uploadFileUri` uploads the file to the target location (using Hadoop DFS's [FileSystem.copyFromLocalFile]({{ hadoop.api }}/org/apache/hadoop/fs/FileSystem.html#copyFromLocalFile-boolean-boolean-org.apache.hadoop.fs.Path-org.apache.hadoop.fs.Path-)) and returns the target URI.

### <span id="uploadFileUri-SparkException"> SparkExceptions

`uploadFileUri` throws a `SparkException` when:

1. Uploading the `uri` fails:

    ```text
    Uploading file [path] failed...
    ```

1. [spark.kubernetes.file.upload.path](configuration-properties.md#spark.kubernetes.file.upload.path) configuration property is not defined:

    ```text
    Please specify spark.kubernetes.file.upload.path property.
    ```

1. `SparkConf` is not defined:

    ```text
    Spark configuration is missing...
    ```

## <span id="renameMainAppResource"> renameMainAppResource

```scala
renameMainAppResource(
  resource: String,
  conf: SparkConf): String
```

`renameMainAppResource` is converted to [spark-internal](overview.md#spark-internal) internal name when the given `resource` [is local and resolvable](#isLocalAndResolvable). Otherwise, `renameMainAppResource` returns the given resource as-is.

`renameMainAppResource` is used when:

* `DriverCommandFeatureStep` is requested for the [base driver container](DriverCommandFeatureStep.md#baseDriverContainer) (for a `JavaMainAppResource` application)

## <span id="isLocalAndResolvable"> isLocalAndResolvable

```scala
isLocalAndResolvable(
  resource: String): Boolean
```

`isLocalAndResolvable` is `true` when the given `resource` is:

1. Not [internal](overview.md#spark-internal)
1. [Uses either file or no URI scheme](#isLocalDependency) (after converting to a well-formed URI)

`isLocalAndResolvable` is used when:

* `KubernetesUtils` is requested to [renameMainAppResource](#renameMainAppResource)
* `BasicDriverFeatureStep` is requested to [getAdditionalPodSystemProperties](BasicDriverFeatureStep.md#getAdditionalPodSystemProperties)

### <span id="isLocalDependency"> isLocalDependency

```scala
isLocalDependency(
  uri: URI): Boolean
```

An input `URI` is a **local dependency** when the scheme is `null` (undefined) or `file`.

## Logging

Enable `ALL` logging level for `org.apache.spark.deploy.k8s.KubernetesUtils` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```text
log4j.logger.org.apache.spark.deploy.k8s.KubernetesUtils=ALL
```

Refer to [Logging](spark-logging.md).
