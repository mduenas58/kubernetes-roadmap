---
tags: [kubernetes, network-policies, security, networking, phase/2]
phase: 2
topic: Network Policies
status: not-started
created: 2026-03-01
---

# Network Policies

By default, Kubernetes allows all pod-to-pod communication. **Network Policies** let you restrict this — like a firewall inside the cluster.

> [!NOTE]
> Network Policies require a **CNI plugin** that supports them. Calico, Cilium, and Weave Net do. Flannel (vanilla) does not.

## Default Behavior

Without any NetworkPolicy, all pods can communicate with each other across all namespaces. This is a significant security risk in multi-tenant or sensitive environments.

## Policy Structure

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}           # {} = applies to ALL pods in namespace
  policyTypes:
    - Ingress
    - Egress
  # no ingress/egress rules = deny all
```

## Common Patterns

### 1. Default Deny All (Start Here)
```yaml
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
  # No rules = deny everything
```
Apply this first, then add explicit allow rules.

### 2. Allow Traffic from Specific Pods
```yaml
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes: [Ingress]
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend     # only backend pods can reach database
      ports:
        - protocol: TCP
          port: 5432
```

### 3. Allow Traffic from Specific Namespace
```yaml
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes: [Ingress]
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              env: production  # only pods from production namespace
```

### 4. Allow Egress to DNS + Specific Services
```yaml
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes: [Egress]
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
      ports:
        - protocol: UDP
          port: 53        # DNS — always needed!
    - to:
        - podSelector:
            matchLabels:
              app: database
      ports:
        - protocol: TCP
          port: 5432
```

> [!WARNING]
> If you block all egress, **DNS will break**. Always allow UDP port 53 to `kube-system`.

### 5. AND vs OR in Selectors
```yaml
# AND — pod must match BOTH selectors
from:
  - namespaceSelector:
      matchLabels:
        env: prod
    podSelector:               # same list item = AND
      matchLabels:
        app: frontend

# OR — traffic from namespace OR from pod
from:
  - namespaceSelector:         # separate list items = OR
      matchLabels:
        env: prod
  - podSelector:
      matchLabels:
        app: frontend
```

## Recommended Starting Policy for a Namespace

```yaml
# 1. Deny all
# 2. Allow pods in same namespace to talk to each other
# 3. Allow DNS
# 4. Add specific cross-namespace rules as needed
```

## Testing

```bash
# Install network debug pod
kubectl run netshoot --image=nicolaka/netshoot -it --rm -- bash

# From inside, test connectivity
curl http://backend-svc:8080
nc -zv database-svc 5432
nslookup kubernetes.default.svc.cluster.local
```

## Resources

| Resource | Link |
|----------|------|
| Network Policies docs | https://kubernetes.io/docs/concepts/services-networking/network-policies |
| Network Policy Editor (visual) | https://editor.networkpolicy.io |
| Calico Network Policy | https://docs.projectcalico.org |

## Practice

- [ ] Install Calico or Cilium on a local cluster
- [ ] Apply default-deny to a namespace and verify all traffic is blocked
- [ ] Add a policy allowing only the frontend to reach the backend
- [ ] Verify DNS still works after locking down egress
- [ ] Use the Network Policy Editor to visualize your policies

---

← [[RBAC]] | Next: [[Resource-Management]] →
