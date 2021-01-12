---
hide:
  - navigation
---

# Demo: Running Spark Examples on Google Kubernetes Engine

This demo shows how to run the official [Spark Examples]({{ spark.doc }}/index.html#running-the-examples-and-shell) to a [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) cluster.

This demo focuses on the ubiquitous SparkPi example, but should let you run the other sample Spark applications too.

```text
./bin/run-example SparkPi 10
```

## Before you begin

Make sure to enable the Kubernetes Engine API (as described in [Deploying a containerized web application](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app#before-you-begin)).

## Building Spark Container Image

```text
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export GCP_CR=eu.gcr.io/${PROJECT_ID}
```

Build and push a Apache Spark base image to [Container Registry](https://cloud.google.com/container-registry/docs) on Google Cloud Platform.

```text
cd $SPARK_HOME
```

```text
./bin/docker-image-tool.sh \
  -b java_image_tag=11-jre-slim \
  -r $GCP_CR \
  -t v3.0.1 \
  build
```

List the images using [docker images](https://docs.docker.com/engine/reference/commandline/images/) command (and some other fancy options).

```text
$ docker images \
  --filter=reference='$GCP_CR/*:*' \
  --format "table {{ '{{' }}.Repository}}\t{{ '{{' }}.Tag}}"
REPOSITORY                                                TAG
eu.gcr.io/spark-on-kubernetes-2021/spark                  v3.0.1
```

Push the container image to the Container Registry so that a GKE cluster can run it in a pod (as described in [Pushing the Docker image to Container Registry](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app#pushing_the_docker_image_to)).

```text
gcloud auth configure-docker
```

```text
./bin/docker-image-tool.sh \
  -b java_image_tag=11-jre-slim \
  -r $GCP_CR \
  -t v3.0.1 \
  push
```

View the image in the repository.

```text
$ gcloud container images list --repository $GCP_CR
NAME
eu.gcr.io/spark-on-kubernetes-2021/spark
```

## Creating Google Kubernetes Engine Cluster

Create a GKE cluster.

```text
export CLUSTER_NAME=spark-examples-cluster
```

The default version of Kubernetes varies per Google Cloud zone and is often older than latest stable release that can be changed using `--cluster-version` option.

```text
gcloud container clusters create $CLUSTER_NAME \
  --cluster-version=1.17.15-gke.800
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

## Running SparkPi on GKE

!!! note
    What follows is a more succinct version of [Demo: Running Spark Application on minikube](running-spark-application-on-minikube.md).

Use the following yaml configuration file to create required resources.

Use the following yaml configuration file (`rbac.yml`) to create required resources.

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

```text
cd $SPARK_HOME
```

```text
export K8S_SERVER=$(kubectl config view --output=jsonpath='{.clusters[].cluster.server}')
export POD_NAME=spark-examples-pod
export SPARK_IMAGE=$GCP_CR/spark:v3.0.1
```

!!! note
    Due to how `run-example` works internally `--jars` option pointing at `spark-examples_2.12-3.0.1.jar` is required.

```text
./bin/run-example \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name $POD_NAME \
  --jars local:///opt/spark/examples/jars/spark-examples_2.12-3.0.1.jar \
  --conf spark.kubernetes.driver.request.cores=400m \
  --conf spark.kubernetes.executor.request.cores=100m \
  --conf spark.kubernetes.container.image=$SPARK_IMAGE \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
   SparkPi 10
```

!!! note
    `spark.kubernetes.*.request.cores` configuration properties were required due to the default machine type of a GKE cluster is too small CPU-wise. You may consider another machine type for a GKE cluster (e.g. `c2-standard-4`).

Open another terminal and watch the pods being created and terminated.

```text
k get po -n spark-demo -w
```

In the end, review the logs.

```text
k logs -n spark-demo $POD_NAME
```

## Cleaning up

Delete the GKE cluster.

```text
gcloud container clusters delete $CLUSTER_NAME --quiet
```

Delete the images.

```text
gcloud container images delete $GCP_CR/spark:v3.0.1 --force-delete-tags --quiet
```
