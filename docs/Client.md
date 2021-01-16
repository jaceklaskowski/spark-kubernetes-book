# Client

`Client` submits a Spark application to [run on Kubernetes](#run) (by creating the driver pod and starting a watcher that monitors and logs the application status).

## Creating Instance

`Client` takes the following to be created:

* <span id="conf"> [KubernetesDriverConf](KubernetesDriverConf.md)
* <span id="builder"> [KubernetesDriverBuilder](KubernetesDriverBuilder.md)
* <span id="kubernetesClient"> Kubernetes' `KubernetesClient`
* <span id="watcher"> [LoggingPodStatusWatcher](LoggingPodStatusWatcher.md)

`Client` is created when:

* `KubernetesClientApplication` is requested to [start](KubernetesClientApplication.md#start)

## <span id="run"> Running Driver Pod

```scala
run(): Unit
```

`run` requests the [KubernetesDriverBuilder](#builder) to [build a KubernetesDriverSpec](KubernetesDriverBuilder.md#buildFromFeatures).

`run` requests the [KubernetesDriverConf](#conf) for the [resourceNamePrefix](KubernetesDriverConf.md#resourceNamePrefix) to be used for the name of the driver's config map:

```text
[resourceNamePrefix]-driver-conf-map
```

`run` [builds a ConfigMap](#buildConfigMap) (with the name and the system properties of the `KubernetesDriverSpec`).

`run` creates a driver container (based on the `KubernetesDriverSpec`) and adds the following:

* `SPARK_CONF_DIR` env var as `/opt/spark/conf`
* `spark-conf-volume` volume mount as `/opt/spark/conf`

`run` creates a driver pod (based on the `KubernetesDriverSpec`) with the driver container and a new `spark-conf-volume` volume for the `ConfigMap`.

`run` requests the [KubernetesClient](#kubernetesClient) to watch for the driver pod (using the [LoggingPodStatusWatcher](#watcher)) and, when available, attaches the `ConfigMap`.

`run` is used when:

* `KubernetesClientApplication` is requested to [start](KubernetesClientApplication.md#start)

### <span id="addDriverOwnerReference"> addDriverOwnerReference

```scala
addDriverOwnerReference(
  driverPod: Pod,
  resources: Seq[HasMetadata]): Unit
```

`addDriverOwnerReference`...FIXME

### <span id="buildConfigMap"> Building ConfigMap

```scala
buildConfigMap(
  configMapName: String,
  conf: Map[String, String]): ConfigMap
```

`buildConfigMap` builds a Kubernetes `ConfigMap` with the given `conf` key-value pairs stored as **spark.properties** data and the given `configMapName` name.

The stored data uses an extra comment:

```text
Java properties built from Kubernetes config map with name: [configMapName]
```

??? tip "Kubernetes Documentation"
    Learn more about [ConfigMaps]({{ k8s.doc }}/concepts/configuration/configmap/) in the official [Kubernetes Documentation]({{ k8s.doc }}/home/).

??? tip "Demo: Spark and Local Filesystem in minikube"
    Learn more about ConfigMaps and volumes in [Demo: Spark and Local Filesystem in minikube](demo/spark-and-local-filesystem-in-minikube.md).
