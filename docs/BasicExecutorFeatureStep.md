# BasicExecutorFeatureStep

`BasicExecutorFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## <span id="executorContainerImage"> spark.kubernetes.executor.container.image

`BasicExecutorFeatureStep` asserts that [spark.kubernetes.executor.container.image](configuration-properties.md#spark.kubernetes.executor.container.image) configuration property is defined or throws a `SparkException`:

```text
Must specify the executor container image
```
