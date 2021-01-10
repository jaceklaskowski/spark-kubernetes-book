# KubernetesDriverEndpoint

`KubernetesDriverEndpoint` is a `DriverEndpoint` ([Apache Spark]({{ book.spark_core }}/scheduler/DriverEndpoint/)).

## <span id="onDisconnected"> Intercepting Executor Lost Event

```scala
onDisconnected(
  rpcAddress: RpcAddress): Unit
```

`onDisconnected` is part of the `RpcEndpoint` (Apache Spark) abstraction.

`onDisconnected` disables the executor known by the `RpcAddress` (found in the `Executors by RpcAddress Registry` registry).
