---
tags: [kubernetes, multi-cluster, cluster-api, federation, phase/4]
phase: 4
topic: Multi-Cluster
status: not-started
created: 2026-03-01
---

# Multi-Cluster Management

Single-cluster architectures eventually hit limits of isolation, geography, or blast radius. Multi-cluster management is the next frontier.

## Why Multiple Clusters?

| Reason | Example |
|--------|---------|
| **Blast radius isolation** | Production crashes don't affect staging |
| **Compliance / data residency** | EU data stays in EU clusters |
| **Geographic distribution** | Latency — clusters in multiple regions |
| **Scale** | Single cluster limits (~5000 nodes) |
| **Team isolation** | Each team owns their cluster |
| **Availability** | Active-active for 99.99%+ uptime |

## Cluster API (CAPI)

Declarative cluster lifecycle management. Define clusters as Kubernetes objects — provision, upgrade, scale, and delete clusters with `kubectl`.

```bash
# Install clusterctl
brew install clusterctl

# Initialize management cluster
clusterctl init --infrastructure aws   # or gcp, azure, docker (local)
```

```yaml
# Cluster definition
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: prod-us-east-1
  namespace: clusters
spec:
  clusterNetwork:
    pods:
      cidrBlocks: ["10.244.0.0/16"]
  infrastructureRef:
    kind: AWSCluster
    name: prod-us-east-1
  controlPlaneRef:
    kind: KubeadmControlPlane
    name: prod-us-east-1-cp
---
apiVersion: controlplane.cluster.x-k8s.io/v1beta1
kind: KubeadmControlPlane
metadata:
  name: prod-us-east-1-cp
spec:
  replicas: 3
  version: v1.29.0
  machineTemplate:
    infrastructureRef:
      kind: AWSMachineTemplate
      name: prod-us-east-1-cp
```

```bash
# Day-2 operations are just kubectl
kubectl scale machinedeployment prod-workers --replicas=10
kubectl patch kubeadmcontrolplane prod-cp --type=merge \
  -p '{"spec":{"version":"v1.30.0"}}'
```

## Multi-Cluster GitOps (ArgoCD)

ArgoCD manages applications across many clusters from a single control plane.

```yaml
# Register clusters
argocd cluster add prod-us-east-1
argocd cluster add prod-eu-west-1

# ApplicationSet — deploy to all clusters matching a label
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: web-app
  namespace: argocd
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            env: production
  template:
    metadata:
      name: '{{name}}-web-app'
    spec:
      source:
        repoURL: https://github.com/myorg/web-app
        path: k8s/overlays/production
        targetRevision: HEAD
      destination:
        server: '{{server}}'
        namespace: web-app
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

## Submariner — Cross-Cluster Networking

Connect pod and Service networks across clusters.

```bash
subctl deploy-broker
subctl join --kubeconfig cluster1.yaml broker-info.subm
subctl join --kubeconfig cluster2.yaml broker-info.subm

# Service export — make a Service available across clusters
kubectl apply -f - <<EOF
apiVersion: multicluster.x-k8s.io/v1alpha1
kind: ServiceExport
metadata:
  name: database
  namespace: production
EOF

# Access from cluster2
curl http://database.production.svc.clusterset.local:5432
```

## KubeFed (Deprecated) → Multicluster Scheduler

KubeFed has been deprecated. Modern alternatives:
- **Liqo** — transparent workload offloading between clusters
- **OCM (Open Cluster Management)** — Red Hat's multi-cluster platform
- **Karmada** — Kubernetes Federation Evolution

## Fleet Management with Fleet (Rancher)

```bash
# Deploy apps to thousands of clusters
helm install fleet-crd fleet/fleet-crd
helm install fleet fleet/fleet
```

## Multi-Cluster Observability

```yaml
# Thanos — multi-cluster Prometheus aggregation
# One Thanos Query layer aggregates data from all cluster Prometheuses

# Grafana — multiple datasources from different clusters
# One Grafana → many Prometheus data sources (one per cluster)
```

## Disaster Recovery Patterns

| Pattern | RTO | RPO | Cost |
|---------|-----|-----|------|
| Backup + restore (Velero) | Hours | Hours | Low |
| Warm standby | Minutes | Minutes | Medium |
| Active-passive | Seconds | Seconds | High |
| Active-active | Near-zero | Near-zero | Highest |

```bash
# Velero — cluster backup and restore
velero install --provider aws --bucket my-backups

# Backup entire cluster
velero backup create full-backup --include-namespaces '*'

# Restore to another cluster
velero restore create --from-backup full-backup
```

## Resources

| Resource | Link |
|----------|------|
| Cluster API | https://cluster-api.sigs.k8s.io |
| ArgoCD ApplicationSet | https://argo-cd.readthedocs.io/en/stable/user-guide/application-set |
| Submariner | https://submariner.io |
| Open Cluster Management | https://open-cluster-management.io |
| Velero | https://velero.io |

## Practice

- [ ] Set up Cluster API with the Docker provider — create a workload cluster programmatically
- [ ] Register two clusters in ArgoCD and deploy the same app to both with ApplicationSet
- [ ] Set up Velero and perform a full backup/restore
- [ ] Connect two local clusters with Submariner and test cross-cluster DNS

---

← [[Security]] | Next: [[Internals]] →
