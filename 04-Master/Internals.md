---
tags: [kubernetes, internals, api-machinery, scheduler, etcd, phase/4]
phase: 4
topic: Internals
status: not-started
created: 2026-03-01
---

# Kubernetes Internals

Understanding how Kubernetes works under the hood separates engineers who operate it from those who can extend and fix it.

## API Machinery

### API Groups and Versioning

```
/api/v1                          → core group (Pods, Services, ConfigMaps…)
/apis/apps/v1                    → apps group (Deployments, StatefulSets…)
/apis/batch/v1                   → batch group (Jobs, CronJobs)
/apis/networking.k8s.io/v1      → networking group
/apis/custom.example.com/v1     → your CRD
```

```bash
# Explore the API
kubectl api-resources
kubectl api-versions
kubectl explain pod.spec.containers
kubectl explain --recursive deployment.spec
```

### API Server Request Pipeline

```
Client → Authentication → Authorization (RBAC) → Admission Controllers → Validation → etcd
```

Admission controllers run in two phases:
1. **Mutating** — can modify the object (inject sidecars, set defaults)
2. **Validating** — can reject the object (enforce policies)

Built-in admission controllers:
- `NamespaceLifecycle` — prevent use of terminating namespaces
- `LimitRanger` — enforce LimitRange
- `ResourceQuota` — enforce ResourceQuota
- `NodeRestriction` — nodes can only modify their own objects
- `PodSecurity` — enforce Pod Security Standards

### Watch Mechanism

Kubernetes controllers and `kubectl -w` use long-polling watches:

```bash
# Raw watch stream
kubectl get pods --watch -v=9     # -v=9 shows HTTP requests

# What you see at low level:
GET /api/v1/namespaces/default/pods?watch=true&resourceVersion=12345
# → returns a stream of events: ADDED, MODIFIED, DELETED
```

The **informer** pattern (used in controllers):
- Initial `List` to sync current state
- Ongoing `Watch` to receive incremental updates
- Local cache to avoid hitting the API server per query

## etcd Deep Dive

```bash
# Inspect etcd directly (on a control plane node)
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/pods/default/my-pod -w json | jq .

# List all keys
etcdctl get / --prefix --keys-only
```

Key prefixes:
```
/registry/pods/
/registry/deployments/
/registry/secrets/
/registry/services/
/registry/namespaces/
```

### Raft Consensus

etcd uses the Raft algorithm for consensus:
- Cluster needs `(n/2) + 1` nodes to be available to accept writes
- 3 nodes → can tolerate 1 failure
- 5 nodes → can tolerate 2 failures
- Never run an even number of etcd members

```bash
# Check etcd cluster health
etcdctl endpoint health --cluster
etcdctl endpoint status --cluster -w table
```

## The Scheduler

The scheduler is a control loop that watches for Pods with `nodeName: ""` and assigns them to nodes.

### Scheduling Stages

1. **Filtering** — find nodes that _can_ run the pod
   - Node has enough CPU/memory (requests)
   - Node satisfies `nodeSelector` and `nodeName`
   - Taints/tolerations allow the pod
   - Pod affinity/anti-affinity constraints met
   - Volume topology requirements satisfied

2. **Scoring** — rank feasible nodes
   - `LeastAllocated` — prefer nodes with more free resources
   - `ImageLocality` — prefer nodes that already have the image
   - `NodeAffinity` — weight by preferred affinity

3. **Binding** — write `nodeName` to the Pod object

### Taints & Tolerations

```yaml
# Taint a node (restrict what can run)
kubectl taint nodes gpu-node gpu=true:NoSchedule

# Pod must tolerate the taint
spec:
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
```

### Node Affinity

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:  # hard
        nodeSelectorTerms:
          - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values: [us-east-1a, us-east-1b]
      preferredDuringSchedulingIgnoredDuringExecution: # soft
        - weight: 1
          preference:
            matchExpressions:
              - key: node-type
                operator: In
                values: [high-memory]
```

### Pod Topology Spread

```yaml
spec:
  topologySpreadConstraints:
    - maxSkew: 1
      topologyKey: topology.kubernetes.io/zone
      whenUnsatisfiable: DoNotSchedule
      labelSelector:
        matchLabels:
          app: web-app
```

## Controller Manager Internals

Each controller is a goroutine running a reconcile loop:

```
Watch → Event Queue → Reconciler → Update API Server
         (rate limited,              (retry on failure)
          deduplication)
```

```bash
# See controllers in action
kubectl get events --sort-by='.lastTimestamp'
kubectl describe replicaset my-rs   # shows controller events
```

## CNI — Container Network Interface

When a Pod is scheduled to a node:
1. kubelet calls the CNI plugin
2. CNI creates a network namespace for the pod
3. CNI creates a virtual ethernet pair (veth)
4. CNI assigns an IP from the Pod CIDR
5. CNI sets up routes so the Pod can reach other Pods

```bash
# Inspect CNI config
ls /etc/cni/net.d/
cat /etc/cni/net.d/10-flannel.conflist

# See pod interfaces
kubectl exec my-pod -- ip addr
kubectl exec my-pod -- ip route
```

## CRI — Container Runtime Interface

kubelet communicates with container runtimes via gRPC over the CRI:

```bash
# Interact with containerd directly (bypassing kubelet)
crictl ps                    # list containers
crictl images                # list images
crictl logs <container-id>
crictl inspect <container-id>
```

## Resources

| Resource | Link |
|----------|------|
| Programming Kubernetes (O'Reilly) | Book |
| Kubernetes source code | https://github.com/kubernetes/kubernetes |
| API Conventions | https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md |
| Kubernetes SIG Architecture docs | https://github.com/kubernetes/community/tree/master/sig-architecture |

## Practice

- [ ] Use `kubectl -v=9` to trace all API calls made by a command
- [ ] Inspect etcd keys directly on a local cluster
- [ ] Manually schedule a Pod by setting `nodeName` — bypass the scheduler
- [ ] Write a minimal controller in Python using the `kubernetes` client library
- [ ] Profile the scheduler using `kubectl get events` to trace pod scheduling decisions

---

← [[Multi-Cluster]] | Next: [[Contributing]] →
