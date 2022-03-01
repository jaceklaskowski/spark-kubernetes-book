---
hide:
  - navigation
---

# Demo: Running PySpark Application on minikube

This demo shows how to deploy a PySpark application to [Kubernetes](../overview.md) (using [minikube](https://minikube.sigs.k8s.io/docs/)).

## Before you begin

It is assumed that you have finished the following:

- [Demo: spark-shell on minikube](spark-shell-on-minikube.md)

## Start Cluster

Unless already started, start minikube.

```text
minikube start
```

## Build PySpark Image

In a separate terminal...

```text
cd $SPARK_HOME
```

!!! tip
    Review `kubernetes/dockerfiles/spark` (in your Spark installation) or `resource-managers/kubernetes/docker/src/main/dockerfiles/spark` (in the Spark source code).

### docker-image-tool

Build and publish the PySpark image. Note `-m` option to point the shell script to use minikube's Docker daemon.

```text
./bin/docker-image-tool.sh \
  -m \
  -t v3.2.1 \
  -p resource-managers/kubernetes/docker/src/main/dockerfiles/spark/bindings/python/Dockerfile \
  build
```

### docker images

Point the shell to minikube's Docker daemon.

```text
eval $(minikube -p minikube docker-env)
```

List the Spark image.

```text
docker images spark-py
```

```text
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
spark-py     v3.2.1    f15c947dea88   32 seconds ago   1.17GB
```

## Build Spark Application Image

Point the shell to minikube's Docker daemon and make sure there is the Spark image (that your Spark application project uses).

```text
eval $(minikube -p minikube docker-env)
```

Use this image in the `Dockerfile` of your Spark application:

```text
FROM spark-py:v{{ spark.version }}
```

Build and push the Docker image of your Spark application project to minikube's Docker repository.

List the images and make sure that the image of your Spark application project is available.

```text
docker images pf-demo
```

```text
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
pf-demo      v1.0.0    7ff4769b11c1   21 seconds ago   1.17GB
```

## Create Kubernetes Resources

Create required Kubernetes resources to run a Spark application.

??? tip "Spark official documentation"
    Learn more from the [Spark official documentation]({{ spark.doc }}/running-on-kubernetes.html#rbac).

Make sure to create the required Kubernetes resources (a service account and a cluster role binding) as without them you surely run into the following exception message:

```text
Forbidden!Configured service account doesn't have access. Service account may have been revoked.
```

### Declaratively

Use the following `rbac.yml` file (from the [Spark on Kubernetes Demos](https://github.com/jaceklaskowski/spark-meetups) project).

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

Create the resources in the Kubernetes cluster.

```text
k create -f rbac.yml
```

!!! tip
    With declarative approach (using `rbac.yml`) cleaning up becomes as simple as `k delete -f rbac.yml`.

## Submit Spark Application to minikube

```text
cd $SPARK_HOME
```

```text
K8S_SERVER=$(k config view --output=jsonpath='{.clusters[].cluster.server}')
export POD_NAME=pf-demo
export IMAGE_NAME=$POD_NAME:v1.0.0
```

Please note the [configuration properties](../configuration-properties.md) (some not really necessary but make the demo easier to guide you through, e.g. [spark.kubernetes.driver.pod.name](../configuration-properties.md#spark.kubernetes.driver.pod.name)).

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name pf-demo \
  --conf spark.kubernetes.container.image=$IMAGE_NAME \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.driver.pod.name=pf-demo \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --conf spark.kubernetes.file.upload.path=/tmp \
  --conf spark.ui.enabled=false \
  --driver-java-options="-Dlog4j.configuration=file:conf/log4j.properties" \
  --conf spark.sql.extensions=pl.japila.spark.sql.MySparkSessionExtension \
  local:///pf-search/demo.py
```

## Accessing web UI

```text
k port-forward $POD_NAME 4040:4040
```

Open http://localhost:4040.

## Accessing Logs

Access the logs of the driver.

```text
k logs -f pf-demo
```

## Spark Application Management

```text
K8S_SERVER=$(kubectl config view --output=jsonpath='{.clusters[].cluster.server}')
```

### Application Status

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --status "spark-demo:$POD_NAME"
```

```text
Application status (driver):
	 pod name: meetup-spark-app
	 namespace: spark-demo
	 labels: spark-app-selector -> spark-0df2be7b2d8d40299e7a406564c9833c, spark-role -> driver
	 pod uid: 30a749a3-1060-49a7-b502-4a054ea33d30
	 creation time: 2021-02-09T13:18:14Z
	 service account name: spark
	 volumes: spark-local-dir-1, spark-conf-volume-driver, spark-token-hqc6k
	 node name: minikube
	 start time: 2021-02-09T13:18:14Z
	 phase: Running
	 container status:
		 container name: spark-kubernetes-driver
		 container image: meetup-spark-app:0.1.0
		 container state: running
		 container started at: 2021-02-09T13:18:15Z
```

### Stop Spark Application

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --kill "spark-demo:$POD_NAME"
```

## Clean Up

Clean up the cluster as described in [Demo: spark-shell on minikube](spark-shell-on-minikube.md#clean-up).

_That's it. Congratulations!_
