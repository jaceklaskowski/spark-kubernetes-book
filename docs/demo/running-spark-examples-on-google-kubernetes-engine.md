---
hide:
  - navigation
---

# Demo: Running Spark Examples on Google Kubernetes Engine

This demo shows how to run the official [Spark Examples]({{ spark.doc }}/index.html#running-the-examples-and-shell) on a Kubernetes cluster on [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine).

This demo focuses on the ubiquitous SparkPi example, but should let you run the other sample Spark applications too.

```text
./bin/run-example SparkPi 10
```

## Before you begin

Open up a Google Cloud project in the [Google Cloud Console](https://console.cloud.google.com/) and enable the Kubernetes Engine API (as described in [Deploying a containerized web application](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app#before-you-begin)).

Review [Demo: Running Spark Examples on minikube](running-spark-application-on-minikube.md) to build a basic understanding of the process of deploying Spark applications to a local Kubernetes cluster using minikube.

## Prepare Spark Base Image

```text
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export GCP_CR=eu.gcr.io/${PROJECT_ID}
```

### Build Spark Image

Build and push a Apache Spark base image to [Container Registry](https://cloud.google.com/container-registry/docs) on Google Cloud Platform.

```text
cd $SPARK_HOME
```

```text
./bin/docker-image-tool.sh \
  -r $GCP_CR \
  -t v{{ spark.version }} \
  build
```

List the images using [docker images](https://docs.docker.com/engine/reference/commandline/images/) command (and some other fancy options).

```text
docker images "$GCP_CR/*" \
  --format "table {{ '{{' }}.Repository}}\t{{ '{{' }}.Tag}}"
```

```text
REPOSITORY                                 TAG
eu.gcr.io/spark-on-kubernetes-2021/spark   v{{ spark.version }}
```

### Push Spark Image to Container Registry

Push the container image to the Container Registry so that a GKE cluster can run it in a pod (as described in [Pushing the Docker image to Container Registry](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app#pushing_the_docker_image_to)).

```text
gcloud auth configure-docker
```

```text
./bin/docker-image-tool.sh \
  -r $GCP_CR \
  -t v{{ spark.version }} \
  push
```

### List Images

Use [gcloud container images list](https://cloud.google.com/sdk/gcloud/reference/container/images/list) to list the Spark image in the repository.

```text
gcloud container images list --repository $GCP_CR
```

```text
NAME
eu.gcr.io/spark-on-kubernetes-2021/spark
```

### List Tags

Use [gcloud container images list-tags](https://cloud.google.com/sdk/gcloud/reference/container/images/list-tags) to list tags and digests for the specified image.

```text
gcloud container images list-tags $GCP_CR/spark
```

```text
DIGEST        TAGS        TIMESTAMP
9a50d1435bbe  v{{ spark.version }}  2021-01-26T13:02:11
```

### Describe Spark Image

Use [gcloud container images describe](https://cloud.google.com/sdk/gcloud/reference/container/images/describe) to list information about the Spark image.

```text
gcloud container images describe $GCP_CR/spark:v{{ spark.version }}
```

```text
image_summary:
  digest: sha256:9a50d1435bbe81dd3a23d3e43c244a0bfc37e14fb3754b68431cbf8510360b84
  fully_qualified_digest: eu.gcr.io/spark-on-kubernetes-2021/spark@sha256:9a50d1435bbe81dd3a23d3e43c244a0bfc37e14fb3754b68431cbf8510360b84
  registry: eu.gcr.io
  repository: spark-on-kubernetes-2021/spark
```

## Create Kubernetes Cluster

```text
export CLUSTER_NAME=spark-examples-cluster
```

The default version of Kubernetes varies per Google Cloud zone and is often older than the latest stable release. A cluster version can be changed using `--cluster-version` option.

Use `gcloud container get-server-config` command to check which Kubernetes versions are available and default in your zone.

```text
gcloud container get-server-config
```

!!! tip
    Use [latest](https://cloud.google.com/kubernetes-engine/versioning#specifying_cluster_version) version alias to use the highest supported Kubernetes version currently available on GKE in the cluster's zone or region.

```text
gcloud container clusters create $CLUSTER_NAME \
  --cluster-version=latest
```

Wait a few minutes before the GKE cluster is ready. In the end, you should see a summary of the cluster.

```text
kubeconfig entry generated for spark-examples-cluster.
NAME                    LOCATION        MASTER_VERSION    MASTER_IP      MACHINE_TYPE  NODE_VERSION      NUM_NODES  STATUS
spark-examples-cluster  europe-west3-b  1.18.15-gke.1100  34.107.115.78  e2-medium     1.18.15-gke.1100  3          RUNNING
```

Review the configuration of the GKE cluster.

```text
k config view
```

Review the cluster's VM instances.

```text
gcloud compute instances list
```

## Run SparkPi on GKE

!!! note
    What follows is a more succinct version of [Demo: Running Spark Application on minikube](running-spark-application-on-minikube.md).

### Create Kubernetes Resources

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

Use `k create` to create the Kubernetes resources.

```text
k create -f k8s/rbac.yml
```

### Submit SparkPi

```text
cd $SPARK_HOME
```

```text
export K8S_SERVER=$(kubectl config view --output=jsonpath='{.clusters[].cluster.server}')
export POD_NAME=spark-examples-pod
export SPARK_IMAGE=$GCP_CR/spark:v{{ spark.version }}
```

!!! important
    For the time being we're going to use `spark-submit` not `run-example`. See [Demo: Running Spark Examples on minikube](running-spark-examples-on-minikube.md#running-sparkpi-on-minikube) for more information.

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name $POD_NAME \
  --class org.apache.spark.examples.SparkPi \
  --conf spark.kubernetes.driver.request.cores=400m \
  --conf spark.kubernetes.executor.request.cores=100m \
  --conf spark.kubernetes.container.image=$SPARK_IMAGE \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar 10
```

!!! note
    `spark.kubernetes.*.request.cores` configuration properties were required due to the default machine type of a GKE cluster is too small CPU-wise. You may consider another machine type for a GKE cluster (e.g. `c2-standard-4`).

Open another terminal and watch the pods being created and terminated. Don't forget about the `spark-demo` namespace.

```text
k get po -n spark-demo -w
```

In the end, review the logs.

```text
k logs -n spark-demo $POD_NAME
```

## Clean Up

Delete the GKE cluster.

```text
gcloud container clusters delete $CLUSTER_NAME --quiet
```

Delete the images.

```text
gcloud container images delete $SPARK_IMAGE --force-delete-tags --quiet
```
