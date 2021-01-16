# DriverServiceFeatureStep

`DriverServiceFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`DriverServiceFeatureStep` takes the following to be created:

* <span id="kubernetesConf"> [KubernetesDriverConf](KubernetesDriverConf.md)
* <span id="clock"> `Clock` (default: `SystemClock`)

`DriverServiceFeatureStep` is created when:

* `KubernetesDriverBuilder` is requested for a [driver pod spec](KubernetesDriverBuilder.md#buildFromFeatures)

## <span id="resolvedServiceName"> Service Name

`DriverServiceFeatureStep` defines **Service Name** based on the [Preferred Service Name](#preferredServiceName) (if fits the name limit) or generates one:

```text
spark-[randomServiceId]-driver-svc
```

`DriverServiceFeatureStep` prints out the following WARN message when generating the service name:

```text
Driver's hostname would preferably be [preferredServiceName], but this is too long (must be <= 63 characters). Falling back to use $shorterServiceName as the driver service's name.
```

The service name is used for the following:

* [Additional System Properties](#getAdditionalPodSystemProperties) (as `spark.driver.host`)
* [Additional Kubernetes Resources](#getAdditionalKubernetesResources)

### <span id="preferredServiceName"> Preferred Service Name

`DriverServiceFeatureStep` uses the [resourceNamePrefix](KubernetesDriverConf.md#resourceNamePrefix) (of the given [KubernetesDriverConf](#kubernetesConf)) with `-driver-svc` postfix as the **Preferred Service Name**.

```text
[resourceNamePrefix]-driver-svc
```

The preferred service name becomes the [resolved service name](#resolvedServiceName) only when shorter than `63` characters.

## <span id="getAdditionalKubernetesResources"> Additional Kubernetes Resources

```scala
getAdditionalKubernetesResources(): Seq[HasMetadata]
```

`getAdditionalKubernetesResources` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#getAdditionalKubernetesResources) abstraction.

`getAdditionalKubernetesResources` defines a Kubernetes service with the [Service Name](#resolvedServiceName).

??? tip "kubectl get services"
    Use `k get services` to list Kubernetes services.

## <span id="getAdditionalPodSystemProperties"> Additional System Properties

```scala
getAdditionalPodSystemProperties(): Map[String, String]
```

`getAdditionalPodSystemProperties` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#getAdditionalPodSystemProperties) abstraction.

`getAdditionalPodSystemProperties` sets the following additional Spark properties:

Name | Value
----------------------------------|---------
 `spark.driver.host`              | [serviceName](#resolvedServiceName)`.`[namespace](KubernetesConf.md#namespace)`.svc`
 `spark.driver.port`              | [driverPort](#driverPort)
 `spark.driver.blockManager.port` &nbsp; &nbsp; &nbsp; &nbsp; | [driverBlockManagerPort](#driverBlockManagerPort)

## <span id="configurePod"> Configuring Driver Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod`...FIXME

## Logging

Enable `ALL` logging level for `org.apache.spark.deploy.k8s.features.DriverServiceFeatureStep` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```text
log4j.logger.org.apache.spark.deploy.k8s.features.DriverServiceFeatureStep=ALL
```

Refer to [Logging](spark-logging.md).
