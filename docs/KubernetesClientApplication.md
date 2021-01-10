# KubernetesClientApplication

`KubernetesClientApplication` is a `SparkApplication` ([Apache Spark]({{ book.spark_core }}/tools/SparkApplication/)) in [Spark on Kubernetes](index.md) in `cluster` deploy mode.

## Creating Instance

`KubernetesClientApplication` takes no arguments to be created.

`KubernetesClientApplication` is created when:

* `SparkSubmit` is requested to launch a Spark application (for [kubernetes in cluster deploy mode]({{ book.spark_core }}/tools/SparkSubmit/#KubernetesClientApplication))

## <span id="start"> Starting Spark Application

```scala
start(
  args: Array[String],
  conf: SparkConf): Unit
```

`start` is part of the `SparkApplication` ([Apache Spark]({{ book.spark_core }}/tools/SparkApplication/#start)) abstraction.

`start` [parses](ClientArguments.md#fromCommandLineArgs) the command-line arguments (`args`) and [runs](#run).

### <span id="run"> run

```scala
run(
  clientArguments: ClientArguments,
  sparkConf: SparkConf): Unit
```

`run` generates a custom Spark Application ID of the format:

```text
spark-[randomUUID-without-dashes]
```

`run` [creates a KubernetesDriverConf](KubernetesConf.md#createDriverConf) (with the given [ClientArguments](ClientArguments.md), `SparkConf` and the custom Spark Application ID).

`run` removes the **k8s://** prefix from the `spark.master` configuration property (which has already been validated by `SparkSubmit` itself).

`run` creates a [LoggingPodStatusWatcherImpl](LoggingPodStatusWatcherImpl.md) (with the `KubernetesDriverConf`).

`run` [creates a KubernetesClient](SparkKubernetesClientFactory.md#createKubernetesClient) (with the master URL, the [namespace](KubernetesConf.md#namespace), and others).

In the end, `run` creates a [Client](Client.md) (with the [KubernetesDriverConf](KubernetesDriverConf.md), a new [KubernetesDriverBuilder](KubernetesDriverBuilder.md), the `KubernetesClient`, and the [LoggingPodStatusWatcherImpl](LoggingPodStatusWatcherImpl.md)) and requests it to [run](Client.md#run).
