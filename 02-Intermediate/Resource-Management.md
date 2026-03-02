---
tags: [kubernetes, resources, hpa, vpa, limits, autoscaling, phase/2]
phase: 2
topic: Resource Management
status: not-started
created: 2026-03-01
---

# Resource Management & Autoscaling

Without resource requests and limits, a single misbehaving pod can take down a node or starve other workloads.

## Requests vs Limits

```yaml
resources:
  requests:
    cpu: "100m"       # 100 millicores = 0.1 vCPU
    memory: "128Mi"   # guaranteed minimum
  limits:
    cpu: "500m"       # 0.5 vCPU max (throttled if exceeded)
    memory: "256Mi"   # max (OOMKilled if exceeded)
```

| | Requests | Limits |
|--|---------|--------|
| **Purpose** | Scheduling guarantee | Runtime enforcement |
| **CPU behavior** | Guaranteed share | Throttled (not killed) |
| **Memory behavior** | Guaranteed share | OOMKilled if exceeded |
| **Affects scheduling** | Yes | No |

> [!WARNING]
> Setting memory limits too low is a common cause of `OOMKilled` pod crashes. Start without limits, measure actual usage, then set limits to ~2x the observed peak.

### CPU Units

```
1000m = 1 CPU = 1 vCPU = 1 Core
 500m = 0.5 CPU
 100m = 0.1 CPU
```

### Memory Units

```
1Ki = 1024 bytes
1Mi = 1024 Ki
1Gi = 1024 Mi
128Mi, 256Mi, 512Mi, 1Gi, 2Gi...
```

## QoS Classes

Kubernetes assigns a QoS class to each pod, which determines eviction priority under memory pressure:

| Class | Condition | Eviction Priority |
|-------|-----------|-------------------|
| `Guaranteed` | requests == limits for all containers | Last |
| `Burstable` | requests < limits (at least one container) | Middle |
| `BestEffort` | no requests or limits | First |

```bash
kubectl get pod my-pod -o jsonpath='{.status.qosClass}'
```

## LimitRange

Set default requests/limits for a namespace (so developers don't have to specify them):

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: staging
spec:
  limits:
    - type: Container
      default:
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
      max:
        cpu: "2"
        memory: "1Gi"
```

## ResourceQuota

Cap total resource consumption per namespace:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: staging-quota
  namespace: staging
spec:
  hard:
    pods: "20"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "10"
```

## Horizontal Pod Autoscaler (HPA)

Scale the number of pod replicas based on metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70     # scale up when avg CPU > 70%
    - type: Resource
      resource:
        name: memory
        target:
          type: AverageValue
          averageValue: 200Mi
```

```bash
# Watch HPA decisions
kubectl get hpa -w
kubectl describe hpa web-hpa

# Test: generate load
kubectl run load --image=busybox --rm -it -- sh -c \
  "while true; do wget -q -O- http://web-svc; done"
```

> [!NOTE]
> HPA requires `metrics-server` to be installed. On minikube: `minikube addons enable metrics-server`.

## Vertical Pod Autoscaler (VPA)

Automatically sets container requests/limits based on observed usage.

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  updatePolicy:
    updateMode: "Off"     # Off = recommend only, Auto = apply and restart
```

> [!NOTE]
> VPA and HPA cannot both manage CPU/memory for the same Deployment. Use KEDA for event-driven scaling as an alternative.

## KEDA — Kubernetes Event-Driven Autoscaling

Scale based on external events: queue depth, HTTP request rate, Prometheus metrics, cron schedule.

```bash
helm install keda kedacore/keda --namespace keda --create-namespace
```

## Resources

| Resource | Link |
|----------|------|
| Resource management | https://kubernetes.io/docs/concepts/configuration/manage-resources-containers |
| HPA | https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale |
| VPA | https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler |
| KEDA | https://keda.sh |

## Practice

- [ ] Deploy an app with no resources — observe where it gets scheduled
- [ ] Set requests/limits and observe the QoS class
- [ ] Apply a LimitRange to a namespace and observe defaults being applied
- [ ] Set up HPA with CPU target — generate load and watch scaling
- [ ] Apply VPA in `Off` mode and observe its recommendations

---

← [[Network-Policies]] | Next: [[Helm]] →
