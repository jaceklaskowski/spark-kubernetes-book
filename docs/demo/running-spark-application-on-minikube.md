# Demo: Running Spark Application on minikube

This demo shows how to deploy a Spark application to [Kubernetes](../index.md) (using [minikube](https://minikube.sigs.k8s.io/docs/)).

!!! tip
    Start with [Demo: spark-shell on minikube](spark-shell-on-minikube.md).

## Start Cluster

Quoting [Prerequisites](http://spark.apache.org/docs/latest/running-on-kubernetes.html#prerequisites):

> We recommend 3 CPUs and 4g of memory to be able to start a simple Spark application with a single executor.

Let's start minikube with enough resources.

```text
$ minikube start --cpus 4 --memory 8192
😄  minikube v1.16.0 na Darwin 11.1
✨  Automatically selected the docker driver
👍  Starting control plane node minikube in cluster minikube
🔥  Creating docker container (CPUs=4, Memory=8192MB) ...
🐳  Przygotowywanie Kubernetesa v1.20.0 na Docker 20.10.0...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

## Building Spark Application Image

Note that the image the Spark application project's image extends from (using `FROM` command) should be `jaceklaskowski/spark:v3.0.1` as follows:

```text
FROM jaceklaskowski/spark:v3.0.1
```

Point the shell to minikube's Docker daemon.

```text
eval $(minikube -p minikube docker-env)
```

In your Spark application project execute the command to build and push a Docker image to minikube's Docker repository.

```text
sbt clean docker:publishLocal
```

List available images (that should include your Spark application project's docker image, e.g. `spark-docker-example`).

```text
$ docker images
REPOSITORY                                TAG           IMAGE ID       CREATED          SIZE
spark-docker-example                      0.1.0         3bfc4e483561   43 seconds ago   510MB
jaceklaskowski/spark                      v3.0.1        78f2f9f19236   43 minutes ago   504MB
openjdk                                   11-jre-slim   57a8cfbe60f3   4 weeks ago      205MB
kubernetesui/dashboard                    v2.1.0        9a07b5b4bfac   4 weeks ago      226MB
k8s.gcr.io/kube-proxy                     v1.20.0       10cc881966cf   4 weeks ago      118MB
k8s.gcr.io/kube-apiserver                 v1.20.0       ca9843d3b545   4 weeks ago      122MB
k8s.gcr.io/kube-controller-manager        v1.20.0       b9fa1895dcaa   4 weeks ago      116MB
k8s.gcr.io/kube-scheduler                 v1.20.0       3138b6e3d471   4 weeks ago      46.4MB
gcr.io/k8s-minikube/storage-provisioner   v4            85069258b98a   5 weeks ago      29.7MB
k8s.gcr.io/etcd                           3.4.13-0      0369cf4303ff   4 months ago     253MB
k8s.gcr.io/coredns                        1.7.0         bfe3a36ebd25   6 months ago     45.2MB
kubernetesui/metrics-scraper              v1.0.4        86262685d9ab   9 months ago     36.9MB
k8s.gcr.io/pause                          3.2           80d28bedfe5d   10 months ago    683kB
```

## (Optional) Creating Namespace

This step is optional, but gives a better exposure to the Kubernetes-related features supported by Apache Spark.

```text
kubectl create namespace spark-demo
```

!!! tip
    Use `kubens` (from [kubectx](https://github.com/ahmetb/kubectx) project) to switch between Kubernetes namespaces smoothly.

```text
$ kubens spark-demo
Context "minikube" modified.
Active namespace is "spark-demo".
```

## Create Service Account

!!! tip
    Learn more from the [Spark official documentation](http://spark.apache.org/docs/latest/running-on-kubernetes.html#rbac).

Without this step you could face the following exception message:

```text
Forbidden!Configured service account doesn't have access. Service account may have been revoked.
```

Create a service account `spark`.

```text
kubectl create serviceaccount spark
```

Create a cluster role binding `spark-role`.

```text
kubectl create clusterrolebinding spark-role \
  --clusterrole edit \
  --serviceaccount spark-demo:spark
```

## Submitting Spark Application to minikube

```text
cd $SPARK_HOME
```

```text
K8S_SERVER=$(kubectl config view --output=jsonpath='{.clusters[].cluster.server}')
```

```text
./bin/spark-submit \
  --master k8s://$K8S_SERVER \
  --deploy-mode cluster \
  --name spark-docker-example \
  --class meetup.SparkApp \
  --conf spark.kubernetes.container.image=spark-docker-example:0.1.0 \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
  --verbose \
  local:///opt/docker/lib/meetup.spark-docker-example-0.1.0.jar
```

After a few seconds, you should see the following messages:

```text
20/12/14 18:34:59 INFO LoggingPodStatusWatcherImpl: Application status for spark-b1f8840227074b62996f66b915044ee6 (phase: Pending)
20/12/14 18:34:59 INFO LoggingPodStatusWatcherImpl: State changed, new state:
	 pod name: spark-docker-example-3c07aa766251ce43-driver
	 namespace: spark-demo
	 labels: spark-app-selector -> spark-b1f8840227074b62996f66b915044ee6, spark-role -> driver
	 pod uid: a8c06d26-ad8a-4b78-96e1-3e0be00a4da8
	 creation time: 2020-12-14T17:34:58Z
	 service account name: spark
	 volumes: spark-local-dir-1, spark-conf-volume, spark-token-tsd97
	 node name: minikube
	 start time: 2020-12-14T17:34:58Z
	 phase: Running
	 container status:
		 container name: spark-kubernetes-driver
		 container image: spark-docker-example:0.1.0
		 container state: running
		 container started at: 2020-12-14T17:34:59Z
20/12/14 18:35:00 INFO LoggingPodStatusWatcherImpl: Application status for spark-b1f8840227074b62996f66b915044ee6 (phase: Running)
```

And then...

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
$ kubectl get services
NAME                                               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
spark-docker-example-3de43976e3a46fcf-driver-svc   ClusterIP   None         <none>        7078/TCP,7079/TCP,4040/TCP   101s
```

## Reviewing Logs

Find the driver pod.

```text
kubectl get pod
```

Review the logs of the driver pod.

```text

```

## Accessing web UI

Find the driver pod.

```text
kubectl get pod
```

```text
kubectl port-forward [driver-pod-name] 4040:4040
```

## Accessing Logs

Access the logs.

```text
kubectl logs -f [driver-pod-name]
```

## Stopping Cluster

```text
minikube stop
```

Optionally (e.g. to start from scratch next time), delete all of the minikube clusters:

```text
minikube delete --all
```