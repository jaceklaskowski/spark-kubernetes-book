---
hide:
  - navigation
---

# Demo: Running Spark Structured Streaming on minikube

This demo shows how to run a Spark Structured Streaming application on minikube:

1. [spark.kubernetes.submission.waitAppCompletion](../configuration-properties.md#spark.kubernetes.submission.waitAppCompletion) configuration property
1. `spark-submit --status` and `--kill`

## Before you begin

It is assumed that you are familiar with the basics of [Spark on Kubernetes](../overview.md) and the other [demos](index.md).

## Start Cluster

Start minikube.

```text
minikube start
```

## Build Spark Application Image

Make sure you've got a Spark image available in minikube's Docker registry. Learn the steps in [Demo: spark-shell on minikube](spark-shell-on-minikube.md#build-spark-image).

Point the shell to minikube's Docker daemon.

```text
eval $(minikube -p minikube docker-env)
```

List the Spark image. Make sure it matches the version of Spark you want to work with.

```text
docker images spark
```

```text
REPOSITORY   TAG          IMAGE ID       CREATED             SIZE
spark        v{{ spark.version }}   e64950545e8f   About an hour ago   509MB
```

Publish the image of the Spark Structured Streaming application. It is project-dependent, and the project uses [sbt](https://www.scala-sbt.org/) with [sbt-native-packager](https://github.com/sbt/sbt-native-packager) plugin.

```text
sbt clean docker:publishLocal
```

List the images and make sure that the image of your Spark application project is available.

```text
docker images spark-streams-demo
```

```text
REPOSITORY           TAG       IMAGE ID       CREATED         SIZE
spark-streams-demo   0.1.0     20145c134ca9   4 minutes ago   515MB
```

## Submit Spark Application to minikube

```text
cd $SPARK_HOME
```

```text
K8S_SERVER=$(k config view --output=jsonpath='{.clusters[].cluster.server}')
```

Make sure that the Kubernetes resources (e.g. a namespace and a service account) are available in the cluster. Learn more in [Demo: Running Spark Application on minikube](running-spark-application-on-minikube.md#declaratively).

```text
k create -f rbac.yml
```

The name of the pod is going to be based on the name of the container image for demo purposes. Pick what works for you.

```text
export POD_NAME=spark-streams-demo
export IMAGE_NAME=$POD_NAME:0.1.0
```

You may optionally delete all pods (since we use a fixed name for the demo).

```text
k delete po --all
```

One of the differences between streaming and batch Spark applications is that the Spark Structured Streaming application is supposed to never stop. That's why the demo uses [spark.kubernetes.submission.waitAppCompletion](../configuration-properties.md#spark.kubernetes.submission.waitAppCompletion) configuration property.

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name $POD_NAME \
  --class meetup.SparkStreamsApp \
  --conf spark.kubernetes.container.image=$IMAGE_NAME \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --conf spark.kubernetes.submission.waitAppCompletion=false \
  --verbose \
  local:///opt/spark/jars/meetup.spark-streams-demo-0.1.0.jar
```

In the end, you should be given a so-called **submission ID** that you're going to use with `spark-submit` tool (via [K8SSparkSubmitOperation](../K8SSparkSubmitOperation.md) extension).

```text
INFO LoggingPodStatusWatcherImpl: Deployed Spark application spark-streams-demo with submission ID spark-demo:spark-streams-demo into Kubernetes
```

Take a note of it as that is how you are going to monitor the application using Spark's `spark-submit --status` (and possibly kill it with `spark-submit --kill`).

```text
export SUBMISSION_ID=spark-demo:spark-streams-demo
```

Once submitted, observe pods in another terminal. Make sure you use `spark-demo` namespace.

```text
k get po -w
```

## Request Status of Spark Application

Use `spark-submit --status SUBMISSION_ID` to requests the status of the Spark driver in `cluster` deploy mode.

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --status $SUBMISSION_ID
```

You should see something similar to the following:

```text
Submitting a request for the status of submission spark-demo:spark-streams-demo in k8s://https://127.0.0.1:55004.
21/01/18 12:16:27 INFO SparkKubernetesClientFactory: Auto-configuring K8S client using current context from users K8S config file
Application status (driver):
	 pod name: spark-streams-demo
	 namespace: spark-demo
	 labels: spark-app-selector -> spark-46ca76cc77c242509f27af3c506eb1f5, spark-role -> driver
	 pod uid: 034ed206-5804-4e9d-ab68-ec56a7678b65
	 creation time: 2021-01-18T11:09:46Z
	 service account name: spark
	 volumes: spark-local-dir-1, spark-conf-volume, spark-token-888gj
	 node name: minikube
	 start time: 2021-01-18T11:09:46Z
	 phase: Running
	 container status:
		 container name: spark-kubernetes-driver
		 container image: spark-streams-demo:0.1.0
		 container state: running
		 container started at: 2021-01-18T11:09:47Z
```

## Kill Spark Application

In the end, you can `spark-submit --kill` the Spark Structured Streaming application.

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --kill $SUBMISSION_ID
```

You should see something similar to the following:

```text
Submitting a request to kill submission spark-demo:spark-streams-demo in k8s://https://127.0.0.1:55004. Grace period in secs: not set.
```

## Clean Up

Clean up the cluster as described in [Demo: spark-shell on minikube](spark-shell-on-minikube.md#clean-up).

_That's it. Congratulations!_
