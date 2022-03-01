---
hide:
  - navigation
---

# Demo: spark-shell on minikube

This demo shows how to run `spark-shell` on [minikube](https://minikube.sigs.k8s.io/docs/) that touts itself as:

> minikube quickly sets up a local Kubernetes cluster on macOS, Linux, and Windows.

!!! note
    `k` is an alias of `kubectl`.

## Start minikube

Let's start `minikube` with the recommended resources (based on the [Prerequisites]({{ spark.doc }}/running-on-kubernetes.html#prerequisites) in the official documentation of Apache Spark).

```text
minikube start --cpus 4 --memory 8192
```

## Review Cluster Info

### Cluster Info

```text
k cluster-info
```

```text
k config view
```

### Pods

List available pods. There should be none except Kubernetes system pods (and that's the reason for `-A` to include all pods, including system's).

```text
k get po -A
```

```text
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-vqm2f            1/1     Running   0          2m3s
kube-system   etcd-minikube                      1/1     Running   0          2m18s
kube-system   kube-apiserver-minikube            1/1     Running   0          2m18s
kube-system   kube-controller-manager-minikube   1/1     Running   0          2m18s
kube-system   kube-proxy-ms4wd                   1/1     Running   0          2m4s
kube-system   kube-scheduler-minikube            1/1     Running   0          2m18s
kube-system   storage-provisioner                1/1     Running   0          2m18s
```

### Container Images

List available Docker images in minikube's Docker registry.

Point the shell to minikube's Docker daemon.

```text
eval $(minikube -p minikube docker-env)
```

List available container images.

```text
docker images | sort
```

```text
REPOSITORY                                TAG       IMAGE ID       CREATED         SIZE
gcr.io/k8s-minikube/storage-provisioner   v5        6e38f40d628d   11 months ago   31.5MB
k8s.gcr.io/coredns/coredns                v1.8.6    a4ca41631cc7   4 months ago    46.8MB
k8s.gcr.io/etcd                           3.5.1-0   25f8c7f3da61   3 months ago    293MB
k8s.gcr.io/kube-apiserver                 v1.23.3   f40be0088a83   4 weeks ago     135MB
k8s.gcr.io/kube-controller-manager        v1.23.3   b07520cd7ab7   4 weeks ago     125MB
k8s.gcr.io/kube-proxy                     v1.23.3   9b7cc9982109   4 weeks ago     112MB
k8s.gcr.io/kube-scheduler                 v1.23.3   99a3486be4f2   4 weeks ago     53.5MB
k8s.gcr.io/pause                          3.6       6270bb605e12   6 months ago    683kB
kubernetesui/dashboard                    v2.3.1    e1482a24335a   8 months ago    220MB
kubernetesui/metrics-scraper              v1.0.7    7801cfc6d5c0   8 months ago    34.4MB
```

### Kubernetes Dashboard

```text
minikube dashboard
```

## Build Spark Image

Quoting [Submitting Applications to Kubernetes]({{ spark.doc }}/running-on-kubernetes.html#submitting-applications-to-kubernetes) in the official documentation of Apache Spark:

> Spark (starting with version 2.3) ships with a Dockerfile that can be used for this purpose, or customized to match an individual application’s needs. It can be found in the `kubernetes/dockerfiles/` directory.

In a separate terminal...

```text
cd $SPARK_HOME
```

!!! tip
    Review `kubernetes/dockerfiles/spark` (in your Spark installation) or `resource-managers/kubernetes/docker/src/main/dockerfiles/spark` (in the Spark source code).

### docker-image-tool

Build and publish the Spark image. Note `-m` option to point the shell script to use minikube's Docker daemon.

```text
./bin/docker-image-tool.sh \
  -m \
  -t v{{ spark.version }} \
  build
```

??? note "11-jre-slim is the default"
    As of Spark 3.1.1, `java_image_tag` argument is assumed `11-jre-slim`. You can use `-b java_image_tag=11-jre-slim` or similar to specify the Java version.

### docker images

Point the shell to minikube's Docker daemon.

```text
eval $(minikube -p minikube docker-env)
```

List the Spark image.

```text
docker images spark
```

```text
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
spark        v3.2.1    5f31df7ad9ea   About a minute ago   802MB
```

### docker image inspect

Use [docker image inspect](https://docs.docker.com/engine/reference/commandline/image_inspect/) command to display detailed information on the Spark image.

```text
docker image inspect spark:v3.2.1
```

## Create Namespace

This step is optional, but gives a better exposure to the Kubernetes features supported by Apache Spark and is highly recommended.

!!! tip
    Learn more about [Creating a new namespace]({{ k8s.doc }}/tasks/administer-cluster/namespaces/#creating-a-new-namespace) in the official documentation of Kubernetes.

```text
k create ns spark-demo
```

Set `spark-demo` as the default namespace using [kubens](https://github.com/ahmetb/kubectx) tool.

```text
kubens spark-demo
```

## Spark Logging

Enable `ALL` logging level for Kubernetes-related loggers to see what happens inside.

Add the following line to `conf/log4j.properties`:

```text
log4j.logger.org.apache.spark.deploy.k8s=ALL
log4j.logger.org.apache.spark.scheduler.cluster.k8s=ALL
log4j.logger.org.apache.spark.scheduler.cluster.k8s.ExecutorPodsAllocator=INFO
```

Refer to [Logging](../spark-logging.md).

## Launch spark-shell

```text
cd $SPARK_HOME
```

```text
K8S_SERVER=$(k config view --output=jsonpath='{.clusters[].cluster.server}')

./bin/spark-shell \
  --master k8s://$K8S_SERVER \
  --conf spark.kubernetes.container.image=spark:v3.2.1 \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --verbose
```

Soon you should see the Spark prompt similar to the following:

```text
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.2.1
      /_/

Using Scala version 2.12.15 (OpenJDK 64-Bit Server VM, Java 11.0.13)
Type in expressions to have them evaluated.
Type :help for more information.
```

Check out the versions.

```text
scala> println(spark.version)
3.2.1
```

```text
scala> println(sc.master)
k8s://https://127.0.0.1:58193
```

## web UIs

Open web UI of the Spark application at http://localhost:4040/.

Review the pods in the [Kubernetes UI](#kubernetes-dashboard). Make sure to use `spark-demo` namespace.

![Pods](../images/spark-shell-on-minikube-pods.png)

## Scale Executors

Just for some more fun, in `spark-shell`, request two more executors and observe the logs.

```text
sc.requestTotalExecutors(numExecutors = 4, localityAwareTasks = 0, hostToLocalTaskCount = Map.empty)
```

```text
sc.killExecutors(Seq("1", "3"))
```

Review the number of executors at http://localhost:4040/executors/ and in the Kubernetes UI.

## Clean Up

```text
minikube stop
```

Close the terminal.

### Full Clean Up

Optionally you may want to clean up all the application resources (e.g. to start from scratch next time).

Delete all of the minikube clusters.

```text
minikube delete --all --purge
```

Remove minikube's Docker images.

```text
docker rmi $(docker image ls 'gcr.io/k8s-minikube/*' -q)
```

```text
docker image prune --force
```

_That's it. Congratulations!_
