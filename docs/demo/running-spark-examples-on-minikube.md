---
hide:
  - navigation
---

# Demo: Running Spark Examples on minikube

This demo shows how to run the Spark example applications on minikube to advance from [spark-shell on minikube](spark-shell-on-minikube.md) to a more serious deployment (yet with no Spark development).

This demo lets you explore deploying a Spark application (e.g. `SparkPi`) to Kubernetes in `cluster` deploy mode.

## Before you begin

Start up minikube with necessary Kubernetes resources.

Follow the steps in [Demo: spark-shell on minikube](spark-shell-on-minikube.md) and [Demo: Running Spark Application on minikube](running-spark-application-on-minikube.md):

1. [Start minikube](spark-shell-on-minikube.md#start-minikube)
1. [Build Spark Image](spark-shell-on-minikube.md#build-spark-image)
1. [Create Kubernetes Resources](running-spark-application-on-minikube.md#create-kubernetes-resources)

## Running SparkPi on minikube

```text
cd $SPARK_HOME
```

```text
K8S_SERVER=$(k config view --output=jsonpath='{.clusters[].cluster.server}')
```

Let's use an environment variable for the name of the pod to be more "stable" and predictable. It should make viewing logs and restarting Spark examples easier. Just change the environment variable or delete the pod and off you go!

```text
export POD_NAME=a1
```

??? note "WIP: No use of run-example without spark.kubernetes.file.upload.path"
    Using the `run-example` shell script to run the Spark examples will not work unless you define [spark.kubernetes.file.upload.path](../configuration-properties.md#spark.kubernetes.file.upload.path) configuration property. The reason is that `run-example` uses the `spark-examples` jar file that is found locally so `spark-submit` (that is used under the covers) has to upload the locally-available resource file to be available in a cluster.

    We'll get to it later. Consider it a work in progress.

    ```text
    ./bin/run-example \
      --master k8s://$K8S_SERVER \
      --deploy-mode cluster \
      --name $POD_NAME \
      --jars local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar \
      --conf spark.kubernetes.container.image=spark:v{{ spark.version }} \
      --conf spark.kubernetes.driver.pod.name=$POD_NAME \
      --conf spark.kubernetes.namespace=spark-demo \
      --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
      --verbose \
      SparkPi 10
    ```

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name $POD_NAME \
  --class org.apache.spark.examples.SparkPi \
  --conf spark.kubernetes.container.image=spark:v{{ spark.version }} \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar 10
```

In another terminal, use `k get po -w` to watch the pods of the driver and executors.

```text
k get po -w
```

Review the logs of the driver (as long as the driver pod is up and running).

```text
k logs $POD_NAME
```

For repeatable `SparkPi` executions, delete the driver pod using `k delete pod` command.

```text
k delete po $POD_NAME
```

!!! tip "spark.kubernetes.executor.deleteOnTermination Configuration Property"
    Use [spark.kubernetes.executor.deleteOnTermination](../configuration-properties.md#spark.kubernetes.executor.deleteOnTermination) configuration property to keep executor pods available once a Spark application is finished (e.g. for examination).

## Using Pod Templates

[spark.kubernetes.driver.podTemplateFile](../configuration-properties.md#spark.kubernetes.driver.podTemplateFile) configuration property allows to define a template file for driver pods (e.g. for multi-container pods).

!!! note "spark.kubernetes.executor.podTemplateFile Configuration Property"
    Use [spark.kubernetes.executor.podTemplateFile](../configuration-properties.md#spark.kubernetes.executor.podTemplateFile) configuration property for the template file of executor pods.

### Pod Template

The following is a very basic pod template file. It is incorrect Spark-wise though (as it does not really allow submitting Spark applications) but does allow playing with pod templates.

```text
spec:
  containers:
  - name: spark
    image: busybox
    command: ['sh', '-c', 'echo "Hello, Spark on Kubernetes!"']
```

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --conf spark.kubernetes.driver.podTemplateFile=pod-template.yml \
  --name $POD_NAME \
  --class org.apache.spark.examples.SparkPi \
  --conf spark.kubernetes.container.image=spark:v{{ spark.version }} \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar 10
```

```text
k logs $POD_NAME
```

```text
Hello, Spark on Kubernetes!
```

## Container Resources

In [cluster](../overview.md#cluster-deploy-mode) deploy mode, Spark on Kubernetes may use extra Non-Heap Memory Overhead in [memory requirements](../BasicDriverFeatureStep.md#driverMemoryWithOverheadMiB) of the driver pod (based on [spark.kubernetes.memoryOverheadFactor](../configuration-properties.md#spark.kubernetes.memoryOverheadFactor) configuration property).

```text
k get po $POD_NAME -o=jsonpath='{.spec.containers[0].resources}' | jq
```

```text
$ k get po $POD_NAME -o=jsonpath='{.spec.containers[0].resources}' | jq
{
  "limits": {
    "memory": "1408Mi"
  },
  "requests": {
    "cpu": "1",
    "memory": "1408Mi"
  }
}
```

Note that this extra memory requirements could be part of a pod template.

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --driver-memory 1g \
  --conf spark.kubernetes.memoryOverheadFactor=0.5 \
  --name $POD_NAME \
  --class org.apache.spark.examples.SparkPi \
  --conf spark.kubernetes.container.image=spark:v{{ spark.version }} \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar 10
```

```text
k get po $POD_NAME -o=jsonpath='{.spec.containers[0].resources}' | jq
```

```text
$ k get po $POD_NAME -o=jsonpath='{.spec.containers[0].resources}' | jq
{
  "limits": {
    "memory": "1536Mi"
  },
  "requests": {
    "cpu": "1",
    "memory": "1536Mi"
  }
}
```
