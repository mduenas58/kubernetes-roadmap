---
tags: [kubernetes, networking, services, ingress, dns, phase/1]
phase: 1
topic: Networking
status: not-started
created: 2026-03-01
---

# Kubernetes Networking — Services & Ingress

## The Kubernetes Networking Model

Three rules that everything else is built on:
1. Every Pod gets a **unique IP** within the cluster
2. Pods can communicate with any other Pod **without NAT**
3. Nodes can communicate with all Pods **without NAT**

This flat network model is implemented by the **CNI plugin** (Calico, Flannel, Cilium, etc.).

## Services

A Service gives a **stable IP and DNS name** to a dynamic set of Pods (selected by labels). Without Services, you'd need to track Pod IPs that change on every restart.

### ClusterIP (default)
- Internal cluster IP only — not reachable from outside
- Use for: communication between services inside the cluster

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  type: ClusterIP
  selector:
    app: web-app      # routes to Pods with this label
  ports:
    - port: 80        # Service port
      targetPort: 8080 # container port
```

### NodePort
- Exposes the Service on a static port on **every Node** (range: 30000–32767)
- Use for: dev/testing, non-cloud environments

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080   # optional, assigned automatically if omitted
```

### LoadBalancer
- Provisions a **cloud load balancer** (AWS ELB, GCP CLB, etc.)
- Use for: production workloads on managed clusters
- Creates a NodePort and ClusterIP automatically

### ExternalName
- Maps a Service to a DNS name (no proxying)
- Use for: aliasing external services inside the cluster

### Headless Service
- `clusterIP: None` — no virtual IP, returns Pod IPs directly via DNS
- Required for StatefulSets so each pod has its own DNS record
- `db-0.db-svc.default.svc.cluster.local`

## Cluster DNS (CoreDNS)

Every Service gets a DNS record:
```
<service>.<namespace>.svc.cluster.local
```

From within the same namespace, just use `<service>`. From other namespaces, use `<service>.<namespace>`.

```bash
# Test DNS from inside a pod
kubectl run tmp --image=busybox --rm -it -- nslookup web-svc
kubectl run tmp --image=busybox --rm -it -- wget -qO- http://web-svc
```

## Ingress

A Service exposes a single app. **Ingress** is an L7 HTTP router: one entry point for many services, with path/host-based routing, TLS termination, and more.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-svc
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-svc
                port:
                  number: 80
```

**Ingress requires an Ingress Controller** (nginx-ingress, Traefik, etc.) — Kubernetes does not include one by default.

```bash
# Install nginx ingress on minikube
minikube addons enable ingress
```

## Service vs Ingress Decision

```
Internal traffic only?          → ClusterIP
Simple external access / dev?   → NodePort
Production on cloud?            → LoadBalancer
Multiple services / TLS / HTTP? → Ingress + ClusterIP Services
```

## Resources

| Resource | Link |
|----------|------|
| Services docs | https://kubernetes.io/docs/concepts/services-networking/service |
| Ingress docs | https://kubernetes.io/docs/concepts/services-networking/ingress |
| Nginx Ingress | https://kubernetes.github.io/ingress-nginx |

## Practice

- [ ] Deploy two apps and connect them using ClusterIP Services — verify DNS resolution
- [ ] Expose an app with a NodePort and access it from your laptop browser
- [ ] Install an Ingress Controller and route two paths to two different Services
- [ ] Add TLS to an Ingress using a self-signed cert stored as a Secret
- [ ] Use a headless Service with a StatefulSet — observe per-pod DNS records

---

← [[Workloads]] | Next: [[Configuration]] →
