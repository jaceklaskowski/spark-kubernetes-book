# KubernetesFeatureConfigStep

`KubernetesFeatureConfigStep` is an [abstraction](#contract) of [Kubernetes pod features](#implementations) for drivers and executors.

## Contract

### <span id="configurePod"> Configuring Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

Used when:

* `KubernetesDriverBuilder` is requested to [buildFromFeatures](KubernetesDriverBuilder.md#buildFromFeatures)
* `KubernetesExecutorBuilder` is requested to [buildFromFeatures](KubernetesExecutorBuilder.md#buildFromFeatures)

### <span id="getAdditionalKubernetesResources"> Additional Kubernetes Resources

```scala
getAdditionalKubernetesResources(): Seq[HasMetadata]
```

Additional Kubernetes resources

Default: `empty`

Used when:

* `KubernetesDriverBuilder` is requested to [buildFromFeatures](KubernetesDriverBuilder.md#buildFromFeatures)

### <span id="getAdditionalPodSystemProperties"> Additional System Properties

```scala
getAdditionalPodSystemProperties(): Map[String, String]
```

Additional system properties of a driver pod (to be used for a [ConfigMap](Client.md#buildConfigMap))

Default: `empty`

Used when:

* `KubernetesDriverBuilder` is requested to [build a KubernetesDriverSpec](KubernetesDriverBuilder.md#buildFromFeatures)

## Implementations

* [BasicDriverFeatureStep](BasicDriverFeatureStep.md)
* [BasicExecutorFeatureStep](BasicExecutorFeatureStep.md)
* [DriverCommandFeatureStep](DriverCommandFeatureStep.md)
* DriverKubernetesCredentialsFeatureStep
* [DriverServiceFeatureStep](DriverServiceFeatureStep.md)
* [EnvSecretsFeatureStep](EnvSecretsFeatureStep.md)
* [ExecutorKubernetesCredentialsFeatureStep](ExecutorKubernetesCredentialsFeatureStep.md)
* HadoopConfDriverFeatureStep
* KerberosConfDriverFeatureStep
* [LocalDirsFeatureStep](LocalDirsFeatureStep.md)
* [MountSecretsFeatureStep](MountSecretsFeatureStep.md)
* [MountVolumesFeatureStep](MountVolumesFeatureStep.md)
* [PodTemplateConfigMapStep](PodTemplateConfigMapStep.md)
