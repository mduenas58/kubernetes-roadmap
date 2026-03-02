---
tags: [kubernetes, architecture, control-plane, phase/1]
phase: 1
topic: Architecture
status: not-started
created: 2026-03-01
---

# Kubernetes Architecture

Understanding what happens when you run `kubectl apply` is the foundation for debugging anything in Kubernetes.

## The Big Picture

```
┌──────────────────────────────────────────────────────┐
│                    Control Plane                     │
│  ┌────────────┐  ┌─────────┐  ┌──────────────────┐  │
│  │ API Server │  │  etcd   │  │    Scheduler     │  │
│  └────────────┘  └─────────┘  └──────────────────┘  │
│  ┌──────────────────────────────────────────────┐    │
│  │         Controller Manager                   │    │
│  └──────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────┘
          │              │              │
    ┌─────┴────┐   ┌─────┴────┐  ┌─────┴────┐
    │  Node 1  │   │  Node 2  │  │  Node 3  │
    │ ┌──────┐ │   │ ┌──────┐ │  │ ┌──────┐ │
    │ │kubelet│ │   │ │kubelet│ │  │ │kubelet│ │
    │ └──────┘ │   │ └──────┘ │  │ └──────┘ │
    │ ┌──────┐ │   │ ┌──────┐ │  │ ┌──────┐ │
    │ │kube- │ │   │ │kube- │ │  │ │kube- │ │
    │ │proxy │ │   │ │proxy │ │  │ │proxy │ │
    │ └──────┘ │   │ └──────┘ │  │ └──────┘ │
    └──────────┘   └──────────┘  └──────────┘
```

## Control Plane Components

### kube-apiserver
- The **only entry point** for all cluster state changes
- Validates and persists objects to etcd
- Serves the REST API consumed by `kubectl` and all controllers
- Stateless — multiple replicas for HA

### etcd
- Distributed **key-value store** — the source of truth for all cluster state
- Only the API server talks to etcd directly
- Understanding: if etcd dies and has no backup, the cluster state is gone
- Data: all Kubernetes objects (Pods, Services, Deployments, Secrets…)

### kube-scheduler
- Watches for **unbound Pods** (no `nodeName` set)
- Selects the best Node based on: resource requests, affinity/anti-affinity, taints/tolerations
- Writes the node name back to the Pod object via the API server — does NOT start the Pod

### kube-controller-manager
- Runs all built-in **control loops** (controllers)
- Each controller watches a resource type and reconciles actual state → desired state
- Examples:
  - **ReplicaSet controller** — ensures N pods are running
  - **Deployment controller** — manages rolling updates
  - **Node controller** — marks nodes as unhealthy after timeout
  - **Endpoints controller** — keeps Service endpoints up to date

### cloud-controller-manager
- Handles cloud-provider-specific logic (LoadBalancer creation, node lifecycle)
- Separate from `kube-controller-manager` to keep the core provider-agnostic

## Node Components

### kubelet
- Runs on **every node**
- Receives PodSpecs from the API server and ensures containers are running
- Reports node and pod status back to the API server
- Talks to the container runtime via CRI (Container Runtime Interface)

### kube-proxy
- Runs on **every node**
- Implements Service networking — programs `iptables` (or IPVS) rules so Service IPs route to Pod IPs
- Does NOT proxy actual traffic (despite the name) — it programs the kernel to do it

### Container Runtime
- Runs containers: `containerd`, `CRI-O`
- Implements the **CRI** so kubelet can talk to it
- Pulls images, creates containers, manages lifecycle

## How kubectl apply Works (End to End)

```
1. kubectl serializes manifest to JSON, sends to kube-apiserver (REST POST/PUT)
2. API server authenticates & authorizes the request
3. API server validates the object (admission controllers run here)
4. API server persists the object to etcd
5. Controller sees the new object via watch, reconciles state
   (e.g. ReplicaSet controller creates Pods)
6. Scheduler sees unbound Pod, picks a node, writes nodeName
7. kubelet on that node sees Pod assigned to it, calls container runtime
8. Container runtime pulls image, creates container
9. kubelet reports Pod as Running back to API server
```

## Resources

| Resource | Type |
|----------|------|
| Kubernetes.io – Cluster Architecture | https://kubernetes.io/docs/concepts/architecture |
| Kubernetes in Action (Lukša) – Ch. 1 | Book |
| "Kubernetes the Hard Way" (Kelsey Hightower) | GitHub Tutorial |

## Practice

- [ ] Draw the architecture from memory
- [ ] Run `kubectl get componentstatuses` (or `kubectl get cs`)
- [ ] Inspect etcd contents with `etcdctl` on a local cluster
- [ ] Watch what happens in real time: `kubectl get pods -w` while applying a Deployment

---

← [[index|Phase 1 Index]] | Next: [[Pods]] →
