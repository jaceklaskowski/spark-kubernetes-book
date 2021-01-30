---
hide:
  - navigation
---

# Demo: Using Persistent Disk for Checkpoint Location in Spark Structured Streaming on Google Kubernetes Engine

This demo is a follow-up to [Demo: Running Spark Structured Streaming on minikube](running-spark-structured-streaming-on-minikube.md) and is going to show the steps to use a persistent disk for a checkpoint location in a Spark Structured Streaming application on Google Kubernetes Engine.

```text
.option("checkpointLocation", "gs://spark-checkpoint-location/")
```

## Before you begin

It is assumed that you have finished the following:

1. [Demo: Running Spark Structured Streaming on minikube](running-spark-structured-streaming-on-minikube.md)
1. [Demo: Running Spark Examples on Google Kubernetes Engine](running-spark-examples-on-google-kubernetes-engine.md)

## Build Spark Application Image

```text
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export GCP_CR=eu.gcr.io/${PROJECT_ID}
```

List images and make sure that the Spark image is available.

```text
gcloud container images list --repository $GCP_CR
```

```text
gcloud container images list-tags $GCP_CR/spark
```

Build the Spark application image and test it locally.

```text
sbt meetup-app-deps/docker:publishLocal spark-streams-demo/docker:publishLocal
```

Once satisifed, push it to the [Container Registry](https://cloud.google.com/container-registry/docs) on Google Cloud Platform.

Use [docker tag](https://docs.docker.com/engine/reference/commandline/tag/) followed by [docker image push](https://docs.docker.com/engine/reference/commandline/image_push/).

```text
docker tag spark-streams-demo:0.1.0 $GCP_CR/spark-streams-demo:0.1.0
```

```text
docker push $GCP_CR/spark-streams-demo:0.1.0
```

List the available images.

```text
gcloud container images list --repository $GCP_CR
```

```text
NAME
eu.gcr.io/spark-on-kubernetes-2021/spark
eu.gcr.io/spark-on-kubernetes-2021/spark-streams-demo
```

## Create Kubernetes Cluster

Create a Kubernetes cluster as described in [Demo: Running Spark Examples on Google Kubernetes Engine](running-spark-examples-on-google-kubernetes-engine.md#create-kubernetes-cluster).

```text
export CLUSTER_NAME=spark-examples-cluster
```

```text
gcloud container clusters create $CLUSTER_NAME \
  --cluster-version=latest
```

Wait a few minutes before the GKE cluster is ready.

## Create Cloud Storage Bucket

Quoting [Connecting to Cloud Storage buckets](https://cloud.google.com/compute/docs/disks/gcs-buckets):

> [Cloud Storage](https://cloud.google.com/storage) is a flexible, scalable, and durable storage option for your virtual machine instances. You can read and write files to Cloud Storage buckets from almost anywhere, so you can use buckets as common storage between your instances, App Engine, your on-premises systems, and other cloud services.

!!! tip
    You may want to review [Storage options](https://cloud.google.com/compute/docs/disks) for alternative instance storage options.

```text
export BUCKET_NAME=gs://spark-checkpoint-location/
```

```text
gsutil mb -b on $BUCKET_NAME
```

### List Contents of Bucket

```text
gsutil ls -l $BUCKET_NAME
```

## Run Spark Structured Streaming on GKE

### Create Kubernetes Resources

Create Kubernetes resources as described in [Demo: Running Spark Examples on Google Kubernetes Engine](running-spark-examples-on-google-kubernetes-engine.md#create-kubernetes-resources).

```text
k create -f k8s/rbac.yml
```

### Submit Spark Application

Submit the Spark application to GKE mimicking the steps in [Demo: Running Spark Structured Streaming on minikube](running-spark-structured-streaming-on-minikube.md#submit-spark-application-to-minikube).

```text
export K8S_SERVER=$(kubectl config view --output=jsonpath='{.clusters[].cluster.server}')
export POD_NAME=spark-streams-demo
export SPARK_IMAGE=$GCP_CR/spark-streams-demo:0.1.0
```

You may optionally delete all pods (since we use a fixed name for the demo).

```text
k delete po --all -n spark-demo
```

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name $POD_NAME \
  --class meetup.SparkStreamsApp \
  --conf spark.kubernetes.driver.request.cores=400m \
  --conf spark.kubernetes.executor.request.cores=100m \
  --conf spark.kubernetes.container.image=$SPARK_IMAGE \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --conf spark.kubernetes.submission.waitAppCompletion=false \
  --verbose \
  local:///opt/spark/jars/meetup.spark-streams-demo-0.1.0.jar $BUCKET_NAME
```

Once submitted, observe pods in another terminal. Make sure you use `spark-demo` namespace.

```text
k get po -w -n spark-demo
```

## FIXME

```text
org.apache.hadoop.fs.UnsupportedFileSystemException: No FileSystem for scheme "gs"
```

## Google Cloud Console

Review the Spark application in the [Google Cloud Console](https://console.cloud.google.com/) of the project:

1. [Workloads](https://console.cloud.google.com/kubernetes/workload)
1. [Services & Ingress](https://console.cloud.google.com/kubernetes/discovery)
1. [Configuration](https://console.cloud.google.com/kubernetes/config) (make sure to use `spark-demo` namespace)

## Kill Spark Application

In the end, you can `spark-submit --kill` the Spark Structured Streaming application.

```text
export SUBMISSION_ID=spark-demo:$POD_NAME
```

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --kill $SUBMISSION_ID
```

## Clean Up

Delete the bucket.

```text
gsutil rm -r $BUCKET_NAME
```

Delete cluster resources as described in [Demo: Running Spark Examples on Google Kubernetes Engine](running-spark-examples-on-google-kubernetes-engine.md#clean-up).

_That's it. Congratulations!_

## References

* [How to use Google Cloud Storage for checkpoint location in streaming query?](https://stackoverflow.com/q/56152492/1305344)
* [Quickstart: Using the gsutil tool](https://cloud.google.com/storage/docs/quickstart-gsutil)