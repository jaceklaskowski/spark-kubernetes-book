---
hide:
  - navigation
---

# Demo: Deploying Spark Application to Google Kubernetes Engine

This demo shows the steps to deploy a Spark application to a [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) cluster.

## Before you begin

Make sure to enable the Kubernetes Engine API (as described in [Deploying a containerized web application](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app#before-you-begin)).

## Building Container Image

```text
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export GCP_REPO=eu.gcr.io/${PROJECT_ID}
```

Build and push a Apache Spark base image to [Container Registry](https://cloud.google.com/container-registry/docs) on Google Cloud Platform.

```text
cd $SPARK_HOME
```

```text
./bin/docker-image-tool.sh \
  -b java_image_tag=11-jre-slim \
  -r $GCP_REPO \
  -t v3.0.1 \
  build
```

```text
docker images
```

```text
./bin/docker-image-tool.sh \
  -b java_image_tag=11-jre-slim \
  -r $GCP_REPO \
  -t v3.0.1 \
  push
```

View the image in the repository.

```text
$ gcloud container images list --repository $GCP_REPO
NAME
eu.gcr.io/spark-on-kubernetes-2021/spark
```

Build the Spark application image.

```text
sbt clean docker:publishLocal
```

List the images using [docker images](https://docs.docker.com/engine/reference/commandline/images/) command (and some other fancy options).

```text
$ docker images \
  --filter=reference='$GCP_REPO/*:*' \
  --format "table {{ '{{' }}.Repository}}\t{{ '{{' }}.Tag}}"
REPOSITORY                                                TAG
eu.gcr.io/spark-on-kubernetes-2021/spark-docker-example   0.1.0
eu.gcr.io/spark-on-kubernetes-2021/spark                  v3.0.1
```

## Pushing Image to Container Registry

Upload the image to a registry so that your GKE cluster can download and run the container image (as described in [Pushing the Docker image to Container Registry](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app#pushing_the_docker_image_to)).

```text
gcloud auth configure-docker
```

```text
$ sbt docker:publish
...
[info] Built image eu.gcr.io/spark-on-kubernetes-2021/spark-docker-example with tags [0.1.0]
[info] The push refers to repository [eu.gcr.io/spark-on-kubernetes-2021/spark-docker-example]
...
[info] Published image eu.gcr.io/spark-on-kubernetes-2021/spark-docker-example:0.1.0
```

View the images in the repository.

```text
$ gcloud container images list --repository $GCP_REPO
NAME
eu.gcr.io/spark-on-kubernetes-2021/spark
eu.gcr.io/spark-on-kubernetes-2021/spark-docker-example
```

## Creating Google Kubernetes Engine Cluster

Create a GKE cluster to run the Spark application.

```text
export CLUSTER_NAME=spark-demo-cluster
```

```text
gcloud container clusters create $CLUSTER_NAME \
  --cluster-version=1.17.15-gke.800 \
  --machine-type=c2-standard-4
```

Wait a few minutes before the GKE cluster is created and health-checked.

Review the configuration of the GKE cluster.

```text
k config view
```

Review the cluster's VM instances.

```text
gcloud compute instances list
```

## Deploying Spark Application to GKE

Let's deploy the Docker image of the Spark application to the GKE cluster.

!!! note
    What follows is a succinct version of [Demo: Running Spark Application on minikube](running-spark-application-on-minikube.md).

Use the following file to create required resources.

```text
apiVersion: v1
kind: Namespace
metadata:
  name: spark-demo
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: spark
  namespace: spark-demo
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: spark-role
  namespace: spark-demo
subjects:
  - kind: ServiceAccount
    name: spark
    namespace: spark-demo
roleRef:
  kind: ClusterRole
  name: edit
  apiGroup: rbac.authorization.k8s.io
---
```

Create the Kubernetes resources.

```text
k create -f rbac.yml
```

Switch to `spark-demo` namespace.

```text
kubens spark-demo
```

```text
cd $SPARK_HOME
```

```text
export K8S_SERVER=$(kubectl config view --output=jsonpath='{.clusters[].cluster.server}')
export DEMO_POD_NAME=spark-demo-gke
export CONTAINER_IMAGE=$GCP_REPO/spark-docker-example:0.1.0
```

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name $DEMO_POD_NAME \
  --class meetup.SparkApp \
  --conf spark.kubernetes.container.image=$CONTAINER_IMAGE \
  --conf spark.kubernetes.driver.pod.name=$DEMO_POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/docker/lib/meetup.spark-docker-example-0.1.0.jar
```

Watch the pods in another terminal.

```text
k get po -w
```

## Accessing Logs

Access the logs of the driver.

```text
k logs -f $DEMO_POD_NAME
```

## Cleaning up

Delete the GKE cluster.

```text
gcloud container clusters delete $CLUSTER_NAME --quiet
```

Delete the images.

```text
gcloud container images delete $GCP_REPO/spark:v3.0.1 --force-delete-tags --quiet
gcloud container images delete $GCP_REPO/spark-docker-example:0.1.0 --force-delete-tags --quiet
```
