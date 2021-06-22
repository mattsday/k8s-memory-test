# Memory Limits vs Requests

This is a simple demo to show what happens if you set different requests and limits for memory and the scheduler needs to place a workload.

## Procedure

1. Create a Kubernetes cluster with a single node and 4GB RAM
2. Deploy the uneven deployment (`uneven-deployment.yaml`)
3. Look at `kubectl top pods` and other metrics to see memory usage
4. Push the even deployment (`even-deployment.yaml`)
5. Look at the stats - e.g. `kubectl describe`, `kubectl top`, etc

If all went as planned it should have evicted the hogging process and restarted the original with less availalbe memory:

```
NAME                           READY   STATUS    RESTARTS   AGE
even-test-78b756d57f-xqnvz     1/1     Running   0          14s
uneven-test-6c4d46fc88-9qdzb   1/1     Running   0          10s
uneven-test-6c4d46fc88-r9r7t   0/1     Evicted   0          2m6s
```

In this case, as the app (based on stress-ng) will always try and balloon to as much memory as it can, it'll get niced by the scheduler, leaving you with a situation like this:

```
NAME                           READY   STATUS    RESTARTS   AGE
even-test-78b756d57f-xqnvz     1/1     Running   0          2m14s
uneven-test-6c4d46fc88-9qdzb   0/1     Evicted   0          2m10s
uneven-test-6c4d46fc88-hd6rg   0/1     Pending   0          102s
uneven-test-6c4d46fc88-r9r7t   0/1     Evicted   0          4m6s
```

The uneven test will never schedule properly and/or keep crashing until there's enough available memory on the cluster.

Deleting the "even" deployment will allow the uneven to assume more resources, as per its limit:

```
NAME                           READY   STATUS    RESTARTS   AGE
uneven-test-6c4d46fc88-9qdzb   0/1     Evicted   0          6m41s
uneven-test-6c4d46fc88-hd6rg   1/1     Running   0          6m13s
uneven-test-6c4d46fc88-r9r7t   0/1     Evicted   0          8m37s
```

Note: it's possible that this exercise will taint the node with `node.kubernetes.io/memory-pressure` - you can either wait for that to timeout or remove it manually once you have removed the "even" workload.
