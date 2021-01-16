# PodTemplateConfigMapStep

`PodTemplateConfigMapStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`PodTemplateConfigMapStep` takes the following to be created:

* <span id="conf"> [KubernetesConf](KubernetesConf.md)

`PodTemplateConfigMapStep` is created when:

* `KubernetesDriverBuilder` is requested for a [driver pod spec](KubernetesDriverBuilder.md#buildFromFeatures)

## <span id="hasTemplate"> hasTemplate Flag

`PodTemplateConfigMapStep` uses `hasTemplate` flag based on the [spark.kubernetes.executor.podTemplateFile](configuration-properties.md#spark.kubernetes.executor.podTemplateFile) configuration property to control whether or not to [configure a pod](#configurePod), and offer [Additional Pod System Properties](#getAdditionalPodSystemProperties) and [Additional Kubernetes Resources](#getAdditionalKubernetesResources).

## <span id="getAdditionalKubernetesResources"> Additional Kubernetes Resources

```scala
getAdditionalKubernetesResources(): Seq[HasMetadata]
```

`getAdditionalKubernetesResources` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#getAdditionalKubernetesResources) abstraction.

`getAdditionalKubernetesResources`...FIXME

## <span id="getAdditionalPodSystemProperties"> Additional System Properties

```scala
getAdditionalPodSystemProperties(): Map[String, String]
```

`getAdditionalPodSystemProperties` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#getAdditionalPodSystemProperties) abstraction.

`getAdditionalPodSystemProperties`...FIXME

## <span id="configurePod"> Configuring Driver Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod`...FIXME
