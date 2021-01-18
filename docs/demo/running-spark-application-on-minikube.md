---
hide:
  - navigation
---

# Demo: Running Spark Application on minikube

This demo shows how to deploy a Spark application to [Kubernetes](../index.md) (using [minikube](https://minikube.sigs.k8s.io/docs/)).

!!! tip
    Start with [Demo: spark-shell on minikube](spark-shell-on-minikube.md).

## Start Cluster

Unless already started, start minikube.

```text
minikube start --cpus 4 --memory 8192
```

## Build Spark Application Image

Make sure you've got a Spark image available in minikube's Docker registry.

Point the shell to minikube's Docker daemon.

```text
eval $(minikube -p minikube docker-env)
```

List the Spark image.

```text
$ docker images spark
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
spark        v3.0.1    62e5d9af786f   2 minutes ago   505MB
```

Use this image in your Spark application:

```text
FROM spark:v3.0.1
```

In your Spark application project execute the command to build and push a Docker image to minikube's Docker repository.

```text
sbt clean docker:publishLocal
```

List the images and make sure that the image of your Spark application project is available.

```text
$ docker images 'meetup*'
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
meetup-spark-app   0.1.0     3a897a9b6f14   25 seconds ago   516MB
meetup-app-deps    0.1.0     e0d9e14b3437   6 minutes ago    510MB
```

## Create Kubernetes Resources

Create required Kubernetes resources to run a Spark application.

??? tip "Spark official documentation"
    Learn more from the [Spark official documentation](http://spark.apache.org/docs/latest/running-on-kubernetes.html#rbac).

A namespace is optional, but the service account and the cluster role binding with proper permissions would lead to the following exception message:

```text
Forbidden!Configured service account doesn't have access. Service account may have been revoked.
```

### Declaratively

Use the following `rbac.yml` file.

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

### Imperatively

```text
k create ns spark-demo
```

```text
k create serviceaccount spark -n spark-demo
```

```text
k create clusterrolebinding spark-role \
  --clusterrole edit \
  --serviceaccount spark-demo:spark \
  -n spark-demo
```

## Submit Spark Application to minikube

```text
cd $SPARK_HOME
```

```text
K8S_SERVER=$(k config view --output=jsonpath='{.clusters[].cluster.server}')
```

```text
export POD_NAME=meetup-spark-app
export IMAGE_NAME=$POD_NAME:0.1.0
```

Please note the [configuration properties](../configuration-properties.md) (some not really necessary but make the demo easier to guide you through, e.g. [spark.kubernetes.driver.pod.name](../configuration-properties.md#spark.kubernetes.driver.pod.name)).

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name $POD_NAME \
  --class meetup.SparkApp \
  --conf spark.kubernetes.container.image=$IMAGE_NAME \
  --conf spark.kubernetes.driver.pod.name=$POD_NAME \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/docker/lib/meetup.meetup-spark-app-0.1.0.jar
```

If all went fine you should soon see `termination reason: Completed` message.

```text
20/12/14 18:35:06 INFO LoggingPodStatusWatcherImpl: State changed, new state:
	 pod name: spark-docker-example-3c07aa766251ce43-driver
	 namespace: spark-demo
	 labels: spark-app-selector -> spark-b1f8840227074b62996f66b915044ee6, spark-role -> driver
	 pod uid: a8c06d26-ad8a-4b78-96e1-3e0be00a4da8
	 creation time: 2020-12-14T17:34:58Z
	 service account name: spark
	 volumes: spark-local-dir-1, spark-conf-volume, spark-token-tsd97
	 node name: minikube
	 start time: 2020-12-14T17:34:58Z
	 phase: Succeeded
	 container status:
		 container name: spark-kubernetes-driver
		 container image: spark-docker-example:0.1.0
		 container state: terminated
		 container started at: 2020-12-14T17:34:59Z
		 container finished at: 2020-12-14T17:35:05Z
		 exit code: 0
		 termination reason: Completed
20/12/14 18:35:06 INFO LoggingPodStatusWatcherImpl: Application status for spark-b1f8840227074b62996f66b915044ee6 (phase: Succeeded)
20/12/14 18:35:06 INFO LoggingPodStatusWatcherImpl: Container final statuses:


	 container name: spark-kubernetes-driver
	 container image: spark-docker-example:0.1.0
	 container state: terminated
	 container started at: 2020-12-14T17:34:59Z
	 container finished at: 2020-12-14T17:35:05Z
	 exit code: 0
	 termination reason: Completed
```

## Accessing web UI

Find the driver pod (`k get po`)

```text
k port-forward spark-demo-minikube 4040:4040
```

## Accessing Logs

Access the logs of the driver.

```text
k logs -f $POD_NAME
```

## Reviewing Spark Application Configuration (ConfigMap)

```text
k get cm
```

```text
k describe cm [driverPod]-conf-map
```

Describe the driver pod and review volumes (`.spec.volumes`) and volume mounts (`.spec.containers[].volumeMounts`).

```text
k describe po $POD_NAME
```

```text
$ k get po $POD_NAME -o=jsonpath='{.spec.volumes}' | jq
[
  {
    "emptyDir": {},
    "name": "spark-local-dir-1"
  },
  {
    "configMap": {
      "defaultMode": 420,
      "name": "spark-docker-example-f76bf776ec818be5-driver-conf-map"
    },
    "name": "spark-conf-volume"
  },
  {
    "name": "spark-token-24krm",
    "secret": {
      "defaultMode": 420,
      "secretName": "spark-token-24krm"
    }
  }
]
```

```text
$ k get po $POD_NAME -o=jsonpath='{.spec.containers[].volumeMounts}' | jq
[
  {
    "mountPath": "/var/data/spark-b5d0a070-ff9a-41a3-91aa-82059ceba5b0",
    "name": "spark-local-dir-1"
  },
  {
    "mountPath": "/opt/spark/conf",
    "name": "spark-conf-volume"
  },
  {
    "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
    "name": "spark-token-24krm",
    "readOnly": true
  }
]
```

## Spark Application Management

```text
K8S_SERVER=$(kubectl config view --output=jsonpath='{.clusters[].cluster.server}')
```

```text
$ ./bin/spark-submit --status "spark-demo:spark-docker-example-*" --master k8s://$K8S_SERVER
...
Application status (driver):
	 pod name: spark-docker-example-3c07aa766251ce43-driver
	 namespace: spark-demo
	 labels: spark-app-selector -> spark-b1f8840227074b62996f66b915044ee6, spark-role -> driver
	 pod uid: a8c06d26-ad8a-4b78-96e1-3e0be00a4da8
	 creation time: 2020-12-14T17:34:58Z
	 service account name: spark
	 volumes: spark-local-dir-1, spark-conf-volume, spark-token-tsd97
	 node name: minikube
	 start time: 2020-12-14T17:34:58Z
	 phase: Succeeded
	 container status:
		 container name: spark-kubernetes-driver
		 container image: spark-docker-example:0.1.0
		 container state: terminated
		 container started at: 2020-12-14T17:34:59Z
		 container finished at: 2020-12-14T17:35:05Z
		 exit code: 0
		 termination reason: Completed
```

## Listing Services

```text
$ k get services
NAME                                               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
spark-docker-example-3de43976e3a46fcf-driver-svc   ClusterIP   None         <none>        7078/TCP,7079/TCP,4040/TCP   101s
```

## Stopping Cluster

```text
minikube stop
```

Optionally (e.g. to start from scratch next time), delete all of the minikube clusters:

```text
minikube delete --all
```
