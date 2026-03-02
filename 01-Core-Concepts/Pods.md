---
tags: [kubernetes, pods, phase/1]
phase: 1
topic: Pods
status: not-started
created: 2026-03-01
---

# Pods

The Pod is the **smallest deployable unit** in Kubernetes — not a container, but a wrapper around one or more containers that share the same network and storage.

## Pod Anatomy

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  namespace: default
  labels:
    app: my-app
    version: "1.0"
spec:
  containers:
    - name: app
      image: nginx:1.25
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
      env:
        - name: ENV
          value: production
      livenessProbe:
        httpGet:
          path: /healthz
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /ready
          port: 80
        initialDelaySeconds: 3
        periodSeconds: 5
```

## Key Concepts

### Pod Lifecycle
```
Pending → Running → Succeeded / Failed
                  ↘ Unknown (node lost contact)
```
- **Pending**: scheduled but containers not yet started (pulling image, waiting for resources)
- **Running**: at least one container is running
- **Succeeded**: all containers exited with code 0 (for Jobs)
- **Failed**: at least one container exited non-zero

### Multi-Container Pods
Containers in a Pod share:
- **Network namespace** — same IP, can communicate via `localhost`
- **Storage** (if volumes are shared)
- **Lifecycle** — they start and stop together

Common multi-container patterns:
| Pattern | Purpose | Example |
|---------|---------|---------|
| **Sidecar** | Augments the main container | Log shipper, proxy |
| **Init container** | Runs to completion before main starts | DB migration, config fetch |
| **Adapter** | Transforms output format | Metrics adapter |
| **Ambassador** | Proxies external connections | Service mesh proxy (Envoy) |

### Health Probes
| Probe | When it fires | Effect on failure |
|-------|--------------|-------------------|
| `livenessProbe` | Continuously while running | Container restarted |
| `readinessProbe` | Continuously while running | Pod removed from Service endpoints |
| `startupProbe` | During startup only | Container restarted if never succeeds |

### Restart Policy
```yaml
spec:
  restartPolicy: Always     # default — always restart
                # OnFailure — restart only on non-zero exit
                # Never     — never restart
```

### Pod vs Container: Why Not Just Containers?
- Pods allow tightly coupled co-location without merging application code
- The shared network namespace enables the service mesh sidecar pattern
- Init containers solve startup ordering without modifying app code

## Essential kubectl Commands

```bash
# Create from manifest
kubectl apply -f pod.yaml

# List pods
kubectl get pods
kubectl get pods -o wide          # show node and IP
kubectl get pods -w               # watch for changes

# Inspect
kubectl describe pod my-app
kubectl logs my-app
kubectl logs my-app -c sidecar    # specific container
kubectl logs my-app --previous    # logs from crashed container

# Execute commands
kubectl exec -it my-app -- bash
kubectl exec -it my-app -c sidecar -- sh

# Delete
kubectl delete pod my-app
kubectl delete -f pod.yaml
```

## Resources

| Resource | Link |
|----------|------|
| Pod docs | https://kubernetes.io/docs/concepts/workloads/pods |
| Init containers | https://kubernetes.io/docs/concepts/workloads/pods/init-containers |
| Configure probes | https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes |

## Practice

- [ ] Write and apply a Pod manifest with a liveness and readiness probe
- [ ] Run a multi-container Pod — have the sidecar write to a shared volume the main container reads from
- [ ] Kill the main container and observe the restart
- [ ] Use `kubectl exec` to explore the container filesystem
- [ ] Use an init container to wait for a dependency before the main container starts

---

← [[Architecture]] | Next: [[Workloads]] →
