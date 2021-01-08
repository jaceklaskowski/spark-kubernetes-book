# LocalDirsFeatureStep

`LocalDirsFeatureStep` is a [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md).

## Creating Instance

`LocalDirsFeatureStep` takes the following to be created:

* <span id="conf"> [KubernetesConf](KubernetesConf.md)
* <span id="defaultLocalDir"> Default Local Directory (default: `/var/data/spark-[randomUUID]`)

`LocalDirsFeatureStep` is created when:

* `KubernetesDriverBuilder` is requested to [build a driver pod](KubernetesDriverBuilder.md#buildFromFeatures)
* `KubernetesExecutorBuilder` is requested to [build an executor pod](KubernetesExecutorBuilder.md#buildFromFeatures)

## <span id="useLocalDirTmpFs"> spark.kubernetes.local.dirs.tmpfs

`LocalDirsFeatureStep` uses [spark.kubernetes.local.dirs.tmpfs](configuration-properties.md#spark.kubernetes.local.dirs.tmpfs) configuration property when [configuring a pod](#configurePod).

## <span id="configurePod"> Configuring Pod

```scala
configurePod(
  pod: SparkPod): SparkPod
```

`configurePod` is part of the [KubernetesFeatureConfigStep](KubernetesFeatureConfigStep.md#configurePod) abstraction.

`configurePod` finds mount paths of the volume mounts with **spark-local-dir-** prefix name of the input `SparkPod` (_localDirs_).

If there are no local directory mount paths, `configurePod`...FIXME

`configurePod` adds the local directory volumes to a new pod specification (there could be none).

`configurePod` defines `SPARK_LOCAL_DIRS` environment variable as a comma-separated local directories and adds the local directory volume mounts to a new container specification (there could be none).

In the end, `configurePod` creates a new `SparkPod` with the new pod and container.
