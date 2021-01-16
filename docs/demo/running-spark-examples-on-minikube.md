---
hide:
  - navigation
---

# Demo: Running Spark Examples on minikube

This demo shows how to run the Spark example applications on minikube to advance from [spark-shell on minikube](spark-shell-on-minikube.md) to a more serious deployment (yet with no Spark development).

This demo lets you explore deploying a Spark application (e.g. `SparkPi`) to Kubernetes in `cluster` deploy mode.

## Before you begin

Set up minikube with necessary Kubernetes resources. Follow the steps in [spark-shell on minikube](spark-shell-on-minikube.md).

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

```text
./bin/run-example \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name $POD_NAME \
  --jars local:///opt/spark/examples/jars/spark-examples_2.12-3.0.1.jar \
  --conf spark.kubernetes.container.image=spark:v3.0.1 \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
   SparkPi 10
```

```text
k logs $POD_NAME
```

```text
k delete po $POD_NAME
```

## Using Pod Templates

[spark.kubernetes.driver.podTemplateFile](../configuration-properties.md#spark.kubernetes.driver.podTemplateFile) configuration property allows to define a template file for driver pods.

### Pod Template

The following is a very basic pod template file. It is incorrect Spark-wise though as it does not really allow submitting Spark applications, but does allow playing with pod templates.

```text
spec:
  containers:
  - name: spark
    image: busybox
    command: ['sh', '-c', 'echo "Hello, Spark on Kubernetes!"']
```

```text
./bin/run-example \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --conf spark.kubernetes.driver.podTemplateFile=pod-template.yml \
  --name $POD_NAME \
  --jars local:///opt/spark/examples/jars/spark-examples_2.12-3.0.1.jar \
  --conf spark.kubernetes.container.image=spark:v3.0.1 \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
   SparkPi 10
```

```text
$ k logs a1
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

```text
./bin/run-example \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --driver-memory 1g \
  --conf spark.kubernetes.memoryOverheadFactor=0.5 \
  --name $POD_NAME \
  --jars local:///opt/spark/examples/jars/spark-examples_2.12-3.0.1.jar \
  --conf spark.kubernetes.container.image=spark:v3.0.1 \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
   SparkPi 10
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
