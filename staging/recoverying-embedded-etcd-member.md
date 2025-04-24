# Recovering from CrashLoopBackOff in vCluster with Embedded etcd

## Symptoms

One or more vCluster pods are in a `CrashLoopBackOff` state while running in high availability mode.

Embedded etcd fails to start with errors such as:

```
Error running embedded etcd: exit status 2
panic: assertion failed: Page expected to be: 4, but self identifies as 6
```

Logs show etcd connection or quorum-related issues.

## Recovery Steps

### 1\. Scale Down to a Single Replica

Scale down the vCluster to **1 replica** using either of the following methods:

Via Helm values:

```
syncer:
  replicas: 1
```

Or manually by editing the StatefulSet:

```
kubectl edit sts <vcluster-name> -n <vcluster-namespace>
```

Set:

```
spec:
 replicas: 1
```

And ensure the `syncer` container includes the argument:

```
--etcd-replicas=1
```

---

### 2\. Delete Problematic Resources

Remove the corrupted volume and corresponding pod:

```
kubectl delete pvc data-<vcluster-name>-1 -n <vcluster-namespace>
kubectl delete pod <vcluster-name>-1 -n <vcluster-namespace>
```

This clears inconsistent etcd data from the affected replica.

---

### 3\. Verify Single Replica Operation

Ensure the remaining pod is running and the vCluster is functional:

```
kubectl get pods -n <vcluster-namespace>
```

---

### 4\. Scale Back Up to Multiple Replicas

Once the single replica is healthy, scale the vCluster back up to **3 replicas**:

Via Helm values:

```
syncer:
  replicas: 3
```

Or manually by editing the StatefulSet:

```
kubectl edit sts <vcluster-name> -n <vcluster-namespace>
```

Set:

```
spec:
 replicas: 3
```

And ensure the `syncer` container includes the argument:

```
--etcd-replicas=3
```

---

### 5\. Monitor Recovery

Watch the pods and logs to confirm that the etcd cluster re-forms and data sync resumes correctly:

```
kubectl get pods -n <vcluster-namespace> -w
```
