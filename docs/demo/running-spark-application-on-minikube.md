---
hide:
  - navigation
---

# Demo: Running Spark Application on minikube

This demo shows how to deploy a Spark application to [Kubernetes](../overview.md) (using [minikube](https://minikube.sigs.k8s.io/docs/)).

## Before you begin

It is assumed that you have finished the following:

- [Demo: spark-shell on minikube](spark-shell-on-minikube.md)

## Start Cluster

Unless already started, start minikube.

```text
minikube start
```

## Build Spark Application Image

Make sure you've got a Spark image available in minikube's Docker registry.

Point the shell to minikube's Docker daemon and make sure there is the Spark image (that your Spark application project uses).

```text
eval $(minikube -p minikube docker-env)
```

List the Spark image.

```text
docker images spark
```

```text
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
spark        v{{ spark.version }}    b3412e410d67   2 hours ago   524MB
```

Use this image in your Spark application:

```text
FROM spark:v{{ spark.version }}
```

Build and push the Docker image of your Spark application project to minikube's Docker repository. The following command assumes that you use [Spark on Kubernetes Demos](https://github.com/jaceklaskowski/spark-meetups) project.

```text
sbt clean \
    'set Docker/dockerRepository in `meetup-spark-app` := None' \
    meetup-spark-app/docker:publishLocal
```

List the images and make sure that the image of your Spark application project is available.

```text
docker images 'meetup*'
```

```text
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
meetup-spark-app   0.1.0     3a867debc6c0   11 seconds ago   524MB
```

### docker image inspect

Use [docker image inspect](https://docs.docker.com/engine/reference/commandline/image_inspect/) command to display detailed information on the Spark application image.

```text
docker image inspect meetup-spark-app:0.1.0
```

### docker image history

Use [docker image history](https://docs.docker.com/engine/reference/commandline/image_history/) command to show the history of the Spark application image.

```text
docker image history meetup-spark-app:0.1.0
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

Use the following `k8s/rbac.yml` file (from the [Spark on Kubernetes Demos](https://github.com/jaceklaskowski/spark-meetups) project).

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
k create -f k8s/rbac.yml
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
  local:///opt/spark/jars/meetup.meetup-spark-app-0.1.0.jar STOP_THE_SPARKCONTEXT
```

!!! important
    `STOP_THE_SPARKCONTEXT` application argument is to stop the `SparkContext` and the driver. The following commands may not work if executed with the argument since the Spark application is stopped.

    Leave it out if you want to play with the commands that follow.

If all goes fine you should soon see `termination reason: Completed` message.

```text
21/03/08 14:35:35 INFO LoggingPodStatusWatcherImpl: State changed, new state:
	 pod name: meetup-spark-app
	 namespace: spark-demo
	 labels: spark-app-selector -> spark-5eba470d52c64518a555951d011ca785, spark-role -> driver
	 pod uid: d61085f6-5764-4320-b77e-02dcd8334382
	 creation time: 2021-03-08T13:35:25Z
	 service account name: spark
	 volumes: spark-local-dir-1, spark-conf-volume-driver, spark-token-kzmdd
	 node name: minikube
	 start time: 2021-03-08T13:35:25Z
	 phase: Succeeded
	 container status:
		 container name: spark-kubernetes-driver
		 container image: meetup-spark-app:0.1.0
		 container state: terminated
		 container started at: 2021-03-08T13:35:27Z
		 container finished at: 2021-03-08T13:35:35Z
		 exit code: 0
		 termination reason: Completed
21/03/08 14:35:35 INFO LoggingPodStatusWatcherImpl: Application status for spark-5eba470d52c64518a555951d011ca785 (phase: Succeeded)
21/03/08 14:35:35 INFO LoggingPodStatusWatcherImpl: Container final statuses:


	 container name: spark-kubernetes-driver
	 container image: meetup-spark-app:0.1.0
	 container state: terminated
	 container started at: 2021-03-08T13:35:27Z
	 container finished at: 2021-03-08T13:35:35Z
	 exit code: 0
	 termination reason: Completed
21/03/08 14:35:35 INFO LoggingPodStatusWatcherImpl: Application meetup-spark-app with submission ID spark-demo:meetup-spark-app finished
21/03/08 14:35:35 DEBUG LoggingPodStatusWatcherImpl: Stopping watching application spark-5eba470d52c64518a555951d011ca785 with last-observed phase Succeeded
```

## Accessing web UI

```text
k port-forward $POD_NAME 4040:4040
```

Open http://localhost:4040.

## Accessing Logs

Access the logs of the driver.

```text
k logs -f $POD_NAME
```

## Reviewing Spark Application Configuration

### ConfigMap

```text
CONFIG_MAP=$(k get cm -o name | grep spark-drv)

k describe $CONFIG_MAP
```

### Volumes

Describe the driver pod and review volumes (`.spec.volumes`) and volume mounts (`.spec.containers[].volumeMounts`).

```text
k describe po $POD_NAME
```

```text
k get po $POD_NAME -o=jsonpath='{.spec.volumes}' | jq
```

```text
[
  {
    "emptyDir": {},
    "name": "spark-local-dir-1"
  },
  {
    "configMap": {
      "defaultMode": 420,
      "items": [
        {
          "key": "log4j.properties",
          "mode": 420,
          "path": "log4j.properties"
        },
        {
          "key": "spark.properties",
          "mode": 420,
          "path": "spark.properties"
        }
      ],
      "name": "spark-drv-b5cf5b7834f5a32d-conf-map"
    },
    "name": "spark-conf-volume-driver"
  },
  {
    "name": "spark-token-sfqc9",
    "secret": {
      "defaultMode": 420,
      "secretName": "spark-token-sfqc9"
    }
  }
]
```

### Volume Mounts

```text
k get po $POD_NAME -o=jsonpath='{.spec.containers[].volumeMounts}' | jq
```

```text
[
  {
    "mountPath": "/var/data/spark-e32e4d73-af0e-43ce-8ffa-f4b64c642b86",
    "name": "spark-local-dir-1"
  },
  {
    "mountPath": "/opt/spark/conf",
    "name": "spark-conf-volume-driver"
  },
  {
    "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
    "name": "spark-token-sfqc9",
    "readOnly": true
  }
]
```

### Services

```text
k get services
```

```text
NAME                                           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
meetup-spark-app-ab8a3a7834f5a022-driver-svc   ClusterIP   None         <none>        7078/TCP,7079/TCP,4040/TCP   31m
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
