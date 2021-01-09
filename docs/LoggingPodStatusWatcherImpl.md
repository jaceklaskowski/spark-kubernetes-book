# LoggingPodStatusWatcherImpl

`LoggingPodStatusWatcherImpl` is a [LoggingPodStatusWatcher](LoggingPodStatusWatcher.md) that [monitors and logs the application status](#watchOrStop).

## Creating Instance

`LoggingPodStatusWatcherImpl` takes the following to be created:

* <span id="conf"> [KubernetesDriverConf](KubernetesDriverConf.md)

`LoggingPodStatusWatcherImpl` is created when:

* `KubernetesClientApplication` is requested to [start](KubernetesClientApplication.md#start)

## <span id="watchOrStop"> watchOrStop

```scala
watchOrStop(
  sId: String): Unit
```

`watchOrStop` is part of the [LoggingPodStatusWatcher](LoggingPodStatusWatcher.md#watchOrStop) abstraction.

`watchOrStop` uses [spark.kubernetes.submission.waitAppCompletion](configuration-properties.md#spark.kubernetes.submission.waitAppCompletion) configuration property to control whether to wait for the Spark application to complete (`true`) or merely print out the following INFO message to the logs:

```text
Deployed Spark application [appName] with submission ID [sId] into Kubernetes
```

While waiting for the Spark application to complete, `watchOrStop` prints out the following INFO message to the logs:

```text
Waiting for application [appName] with submission ID [sId] to finish...
```

Until [podCompleted](#podCompleted) flag is `true`, `watchOrStop` waits [spark.kubernetes.report.interval](configuration-properties.md#spark.kubernetes.report.interval) configuration property and prints out the following INFO message to the logs:

```text
Application status for [appId] (phase: [phase])
```

Once [podCompleted](#podCompleted) flag is `true`, `watchOrStop` prints out the following INFO messages to the logs:

```text
Container final statuses:

[containersDescription]
```

```text
Application [appName] with submission ID [sId] finished
```

When no [pod](#pod) is available, `watchOrStop` prints out the following INFO message to the logs:

```text
No containers were found in the driver pod.
```

## <span id="eventReceived"> eventReceived

```scala
eventReceived(
  action: Action,
  pod: Pod): Unit
```

`eventReceived` is part of the Kubernetes' `Watcher` abstraction.

`eventReceived` brances off based on the given `Action`:

* For `DELETED` or `ERROR` actions, `eventReceived` [closeWatch](#closeWatch)

* For any other actions, [logLongStatus](#logLongStatus) followed by [closeWatch](#closeWatch) if [hasCompleted](#hasCompleted).

### <span id="logLongStatus"> logLongStatus

```scala
logLongStatus(): Unit
```

`logLongStatus` prints out the following INFO message to the logs:

```text
State changed, new state: [formatPodState|unknown]
```

### <span id="hasCompleted"> hasCompleted

```scala
hasCompleted(): Boolean
```

`hasCompleted` is `true` when the [phase](#phase) is `Succeeded` or `Failed`.

`hasCompleted` is used when:

* `LoggingPodStatusWatcherImpl` is requested to [eventReceived](#eventReceived) (when an action is neither `DELETED` nor `ERROR`)

## <span id="podCompleted"> podCompleted Flag

`LoggingPodStatusWatcherImpl` turns `podCompleted` off when [created](#creating-instance).

Until `podCompleted` is on, `LoggingPodStatusWatcherImpl` waits the [spark.kubernetes.report.interval](configuration-properties.md#spark.kubernetes.report.interval) configuration property and prints out the following INFO message to the logs:

```text
Application status for [appId] (phase: [phase])
```

`podCompleted` turns [podCompleted](#podCompleted) on when [closeWatch](#closeWatch).

## <span id="closeWatch"> closeWatch

```scala
closeWatch(): Unit
```

`closeWatch` turns [podCompleted](#podCompleted) on.

`closeWatch` is used when:

* `LoggingPodStatusWatcherImpl` is requested to [eventReceived](#eventReceived) and [onClose](#onClose)

## Logging

Enable `ALL` logging level for `org.apache.spark.deploy.k8s.submit.LoggingPodStatusWatcherImpl` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```text
log4j.logger.org.apache.spark.deploy.k8s.submit.LoggingPodStatusWatcherImpl=ALL
```

Refer to [Logging](spark-logging.md).
