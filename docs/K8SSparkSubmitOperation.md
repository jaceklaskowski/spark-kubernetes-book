# K8SSparkSubmitOperation

`K8SSparkSubmitOperation` is a `SparkSubmitOperation` ([Apache Spark]({{ book.spark_core }}/tools/SparkSubmitOperation/)) for `spark-submit` for [Spark on Kubernetes](#supports).

`K8SSparkSubmitOperation` is registered with Apache Spark using `META-INF/services/org.apache.spark.deploy.SparkSubmitOperation` service file.

## <span id="kill"> Killing Submission

```scala
kill(
  submissionId: String,
  conf: SparkConf): Unit
```

`kill` is part of the `SparkSubmitOperation` ([Apache Spark]({{ book.spark_core }}/tools/SparkSubmitOperation/#kill)) abstraction.

`kill` prints out the following message to standard error:

```text
Submitting a request to kill submission [submissionId] in [spark.master]. Grace period in secs: [[getGracePeriod] | not set].
```

`kill` creates a `KillApplication` to [execute](#execute) it (with the input `submissionId` and `SparkConf`).

## <span id="printSubmissionStatus"> Displaying Submission Status

```scala
printSubmissionStatus(
  submissionId: String,
  conf: SparkConf): Unit
```

`printSubmissionStatus` is part of the `SparkSubmitOperation` ([Apache Spark]({{ book.spark_core }}/tools/SparkSubmitOperation/#printSubmissionStatus)) abstraction.

`printSubmissionStatus` prints out the following message to standard error:

```text
Submitting a request for the status of submission [submissionId] in [spark.master].
```

`printSubmissionStatus` creates a `ListStatus` to [execute](#execute) it (with the input `submissionId` and `SparkConf`).

## <span id="supports"> Checking Whether Master URL Supported

```scala
supports(
  master: String): Boolean
```

`supports` is part of the `SparkSubmitOperation` ([Apache Spark]({{ book.spark_core }}/tools/SparkSubmitOperation/#supports)) abstraction.

`supports` is `true` when the input `master` starts with **k8s://** prefix.

## <span id="execute"> Executing Operation

```scala
execute(
  submissionId: String,
  sparkConf: SparkConf,
  op: K8sSubmitOp): Unit
```

`execute` is used for [kill](#kill) and [printSubmissionStatus](#printSubmissionStatus).

`execute` [parses the master URL](KubernetesUtils.md#parseMasterUrl) (based on `spark.master` configuration property).

`execute` uses `:` (the colon) to split the given `submissionId` to at most two parts - an optional namespace and a driver pod name. The driver pod name can use a glob pattern as `*` (the star).

`execute` [creates a KubernetesClient](SparkKubernetesClientFactory.md#createKubernetesClient) (with `Submission` client type).

If the pod name uses the glob pattern (with `*`), `execute` requests the `KubernetesClient` for the driver pods (pods with the `spark-role` label with `driver` value) that it then hands over to the given `K8sSubmitOp` to `executeOnGlob` (with the optional namespace).

Otherwise, `execute` requests the given `K8sSubmitOp` to `executeOnPod` with the pod and the optional namespace.

`execute` prints out the following message and exits when the given `submissionId` cannot be split on `:` to two parts:

```text
Submission ID: {[submissionId]} is invalid.
```
