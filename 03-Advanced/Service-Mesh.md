---
tags: [kubernetes, service-mesh, istio, linkerd, mtls, phase/3]
phase: 3
topic: Service Mesh
status: not-started
created: 2026-03-01
---

# Service Mesh — Istio & Linkerd

A service mesh is an infrastructure layer that handles **service-to-service communication** transparently: mTLS, traffic management, observability, and retries — without touching application code.

## How It Works

A sidecar proxy (Envoy, linkerd-proxy) is injected into every Pod. All traffic between services passes through the proxies:

```
Pod A                          Pod B
┌─────────────────┐            ┌─────────────────┐
│ App  │  Envoy  │ ──────────► │ Envoy  │  App  │
│      │  proxy  │             │  proxy │       │
└─────────────────┘            └─────────────────┘
         ▲                               ▲
         └────── Control Plane ──────────┘
                 (Istiod / Linkerd CP)
```

## Istio

The most feature-rich mesh. Steeper learning curve, higher resource overhead.

### Install
```bash
curl -L https://istio.io/downloadIstio | sh -
istioctl install --set profile=demo

# Enable sidecar injection for a namespace
kubectl label namespace production istio-injection=enabled
```

### mTLS — Mutual TLS
```yaml
# Enforce strict mTLS in a namespace
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT    # STRICT = mTLS required, PERMISSIVE = allow both
```

### Traffic Management

**VirtualService** — controls routing:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: web-app
spec:
  hosts: [web-app]
  http:
    - match:
        - headers:
            x-version:
              exact: "v2"
      route:
        - destination:
            host: web-app
            subset: v2
    - route:
        - destination:
            host: web-app
            subset: v1
            weight: 90
        - destination:
            host: web-app
            subset: v2
            weight: 10    # canary: 10% to v2
```

**DestinationRule** — defines subsets and load balancing:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: web-app
spec:
  host: web-app
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
    outlierDetection:
      consecutive5xxErrors: 3
      interval: 30s
      baseEjectionTime: 30s   # circuit breaker
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

### Authorization Policies
```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: backend-authz
  namespace: production
spec:
  selector:
    matchLabels:
      app: backend
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/production/sa/frontend"]
      to:
        - operation:
            methods: ["GET", "POST"]
            paths: ["/api/*"]
```

### Observability

Istio automatically generates:
- **Metrics**: request count, latency, error rate per service
- **Traces**: distributed traces via Jaeger/Zipkin
- **Access logs**: per-request logs

```bash
# Kiali — service mesh dashboard
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml
istioctl dashboard kiali
```

## Linkerd

Simpler, faster, lighter. Written in Rust. Less features than Istio but easier to operate.

```bash
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh
linkerd install --crds | kubectl apply -f -
linkerd install | kubectl apply -f -
linkerd check

# Inject proxy into a namespace
kubectl annotate namespace production linkerd.io/inject=enabled
```

## Istio vs Linkerd

| Feature | Istio | Linkerd |
|---------|-------|---------|
| Proxy | Envoy (C++) | linkerd-proxy (Rust) |
| Resource usage | Higher | Lower |
| Learning curve | Steep | Gentler |
| Traffic management | Extensive | Basic |
| mTLS | Yes | Yes (automatic) |
| Community | Large | Growing |

## When to Use a Service Mesh

Use it when you need:
- mTLS between services (zero-trust networking)
- Fine-grained traffic control (canary, A/B, circuit breaking)
- Uniform observability across all services
- Service-level authorization policies

Don't use it if:
- Your cluster is small and simple
- You're not ready for the operational overhead
- Your apps already handle these concerns

## Resources

| Resource | Link |
|----------|------|
| Istio docs | https://istio.io/docs |
| Linkerd docs | https://linkerd.io/docs |
| Istio in Practice (Calcote) | Book |

## Practice

- [ ] Install Istio on a local cluster with demo profile
- [ ] Enable automatic mTLS and verify with `istioctl` proxy-status
- [ ] Set up a 90/10 traffic split between two versions of an app
- [ ] Trigger a circuit breaker with a fault injection policy
- [ ] Install Kiali and explore the service graph

---

← [[CICD]] | Next: [[Operators]] →
