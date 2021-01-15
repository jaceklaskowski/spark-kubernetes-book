---
hide:
  - navigation
---

# Demo: spark-shell on minikube

This demo shows how to run `spark-shell` on [Kubernetes](../index.md) (using [minikube](https://minikube.sigs.k8s.io/docs/)).

## Minikube

Quoting the [official documentation]({{ spark.doc }}/running-on-kubernetes.html) of Apache Spark:

> Spark (starting with version 2.3) ships with a Dockerfile that can be used for this purpose, or customized to match an individual application’s needs. It can be found in the `kubernetes/dockerfiles/` directory.

## Start Cluster

```text
$ minikube start --cpus 4 --memory 8192
😄  minikube v1.16.0 on Darwin 11.1
✨  Automatically selected the docker driver
👍  Starting control plane node minikube in cluster minikube
🔥  Creating docker container (CPUs=4, Memory=8192MB) ...
🐳  Preparing Kubernetes v1.20.0 on Docker 20.10.0 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

```text
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:55020
KubeDNS is running at https://127.0.0.1:55020/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```text
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/jacek/.minikube/ca.crt
    server: https://127.0.0.1:55020
  name: minikube
contexts:
- context:
    cluster: minikube
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/jacek/.minikube/profiles/minikube/client.crt
    client-key: /Users/jacek/.minikube/profiles/minikube/client.key
```

```text
$ kubectl get po -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-465jt            1/1     Running   0          47s
kube-system   etcd-minikube                      0/1     Running   0          62s
kube-system   kube-apiserver-minikube            1/1     Running   0          62s
kube-system   kube-controller-manager-minikube   0/1     Running   0          62s
kube-system   kube-proxy-67plz                   1/1     Running   0          48s
kube-system   kube-scheduler-minikube            0/1     Running   0          62s
kube-system   storage-provisioner                1/1     Running   1          62s
```

## Accessing Kubernetes Dashboard

```text
$ minikube dashboard
🔌  Enabling dashboard ...
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:55244/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

## Building Spark Images

In a separate terminal...

```text
cd $SPARK_HOME
```

!!! tip
    Review `kubernetes/dockerfiles/spark` (in your Spark installation) or `resource-managers/kubernetes/docker` (in the Spark source code).

Build and publish the Spark images. Note `-m` option to point the shell script to use minikube's Docker daemon.

```shell
./bin/docker-image-tool.sh \
  -m \
  -b java_image_tag=11-jre-slim \
  -r jaceklaskowski \
  -t v3.0.1 \
  build
```

Point the shell to minikube's Docker daemon.

```text
eval $(minikube -p minikube docker-env)
```

List available images.

```text
$ docker images
REPOSITORY                                TAG           IMAGE ID       CREATED         SIZE
jaceklaskowski/spark                      v3.0.1        4561fd7bd264   6 seconds ago   505MB
openjdk                                   11-jre-slim   9088803d66fe   2 days ago      205MB
kubernetesui/dashboard                    v2.1.0        9a07b5b4bfac   4 weeks ago     226MB
k8s.gcr.io/kube-proxy                     v1.20.0       10cc881966cf   5 weeks ago     118MB
k8s.gcr.io/kube-scheduler                 v1.20.0       3138b6e3d471   5 weeks ago     46.4MB
k8s.gcr.io/kube-apiserver                 v1.20.0       ca9843d3b545   5 weeks ago     122MB
k8s.gcr.io/kube-controller-manager        v1.20.0       b9fa1895dcaa   5 weeks ago     116MB
gcr.io/k8s-minikube/storage-provisioner   v4            85069258b98a   6 weeks ago     29.7MB
k8s.gcr.io/etcd                           3.4.13-0      0369cf4303ff   4 months ago    253MB
k8s.gcr.io/coredns                        1.7.0         bfe3a36ebd25   7 months ago    45.2MB
kubernetesui/metrics-scraper              v1.0.4        86262685d9ab   9 months ago    36.9MB
k8s.gcr.io/pause                          3.2           80d28bedfe5d   11 months ago   683kB
```

## Creating Namespace

!!! tip
    Learn more in [Creating a new namespace]({{ k8s.doc }}/tasks/administer-cluster/namespaces/#creating-a-new-namespace).

```text
kubectl create namespace spark-demo
```

```text
$ kubectl get ns
NAME                   STATUS   AGE
default                Active   7m30s
kube-node-lease        Active   7m31s
kube-public            Active   7m31s
kube-system            Active   7m31s
kubernetes-dashboard   Active   5m51s
spark-demo             Active   11s
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
```

Refer to [Logging](../spark-logging.md).

## Launching spark-shell

```text
cd $SPARK_HOME
```

```text
K8S_SERVER=$(kubectl config view --output=jsonpath='{.clusters[].cluster.server}')
```

```text
./bin/spark-shell \
  --master k8s://$K8S_SERVER \
  --conf spark.kubernetes.container.image=jaceklaskowski/spark:v3.0.1 \
  --conf spark.kubernetes.context=minikube \
  --conf spark.kubernetes.namespace=spark-demo \
  --verbose
```

```text
21/01/11 18:01:01 INFO SparkKubernetesClientFactory: Auto-configuring K8S client using context minikube from users K8S config file
21/01/11 18:01:02 DEBUG ExecutorPodsWatchSnapshotSource: Starting watch for pods with labels spark-app-selector=spark-application-1610384461937, spark-role=executor.
21/01/11 18:01:02 DEBUG ExecutorPodsAllocator: Pod allocation status: 0 running, 0 pending, 0 unacknowledged.
21/01/11 18:01:02 INFO ExecutorPodsAllocator: Going to request 2 executors from Kubernetes.
21/01/11 18:01:02 DEBUG ExecutorPodsPollingSnapshotSource: Starting to check for executor pod state every 30000 ms.
21/01/11 18:01:02 DEBUG ExecutorPodsAllocator: Requested executor with id 1 from Kubernetes.
21/01/11 18:01:02 DEBUG ExecutorPodsWatchSnapshotSource: Received executor pod update for pod named spark-shell-28fe8176f264cdab-exec-1, action ADDED
21/01/11 18:01:02 DEBUG ExecutorPodsWatchSnapshotSource: Received executor pod update for pod named spark-shell-28fe8176f264cdab-exec-1, action MODIFIED
21/01/11 18:01:02 DEBUG ExecutorPodsAllocator: Requested executor with id 2 from Kubernetes.
21/01/11 18:01:02 DEBUG ExecutorPodsWatchSnapshotSource: Received executor pod update for pod named spark-shell-28fe8176f264cdab-exec-1, action MODIFIED
21/01/11 18:01:02 DEBUG ExecutorPodsAllocator: Still waiting for 2 executors before requesting more.
21/01/11 18:01:02 DEBUG ExecutorPodsWatchSnapshotSource: Received executor pod update for pod named spark-shell-28fe8176f264cdab-exec-2, action ADDED
21/01/11 18:01:02 DEBUG ExecutorPodsWatchSnapshotSource: Received executor pod update for pod named spark-shell-28fe8176f264cdab-exec-2, action MODIFIED
21/01/11 18:01:02 DEBUG ExecutorPodsWatchSnapshotSource: Received executor pod update for pod named spark-shell-28fe8176f264cdab-exec-2, action MODIFIED
21/01/11 18:01:03 DEBUG ExecutorPodsWatchSnapshotSource: Received executor pod update for pod named spark-shell-28fe8176f264cdab-exec-1, action MODIFIED
21/01/11 18:01:03 DEBUG ExecutorPodsAllocator: Pod allocation status: 1 running, 1 pending, 0 unacknowledged.
21/01/11 18:01:03 DEBUG ExecutorPodsAllocator: Still waiting for 1 executors before requesting more.
21/01/11 18:01:04 DEBUG ExecutorPodsWatchSnapshotSource: Received executor pod update for pod named spark-shell-28fe8176f264cdab-exec-2, action MODIFIED
21/01/11 18:01:04 DEBUG ExecutorPodsAllocator: Pod allocation status: 2 running, 0 pending, 0 unacknowledged.
21/01/11 18:01:04 DEBUG ExecutorPodsAllocator: Current number of running executors is equal to the number of requested executors. Not scaling up further.
21/01/11 18:01:06 INFO KubernetesClusterSchedulerBackend$KubernetesDriverEndpoint: Registered executor NettyRpcEndpointRef(spark-client://Executor) (192.168.68.106:55654) with ID 1
21/01/11 18:01:06 INFO KubernetesClusterSchedulerBackend$KubernetesDriverEndpoint: Registered executor NettyRpcEndpointRef(spark-client://Executor) (192.168.68.106:55655) with ID 2
21/01/11 18:01:07 INFO KubernetesClusterSchedulerBackend: SchedulerBackend is ready for scheduling beginning after reached minRegisteredResourcesRatio: 0.8
Spark context Web UI available at http://192.168.68.106:4040
Spark context available as 'sc' (master = k8s://https://127.0.0.1:55020, app id = spark-application-1610384461937).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.0.1
      /_/

Using Scala version 2.12.10 (OpenJDK 64-Bit Server VM, Java 11.0.9)
Type in expressions to have them evaluated.
Type :help for more information.

scala> spark.version
res0: String = 3.0.1

scala> sc.master
res1: String = k8s://https://127.0.0.1:55020
```

## web UIs

Open web UI of the Spark application at http://localhost:4040/.

Review the pods in the Kubernetes UI. Make sure to switch to `spark-demo` namespace.

![Pods](../images/spark-shell-on-minikube-pods.png)

## Scaling Executors Up and Down

Just for some more fun, in `spark-shell`, request two more executors and observe the logs.

```text
sc.requestTotalExecutors(numExecutors = 4, localityAwareTasks = 0, hostToLocalTaskCount = Map.empty)
```

```text
sc.killExecutors(Seq("1", "3"))
```

Review the number of executors at http://localhost:4040/executors/ and in the Kubernetes UI.

## Stopping Cluster

```text
minikube stop
```

Optionally (e.g. to start from scratch next time), delete all of the minikube clusters:

```text
minikube delete --all
```
