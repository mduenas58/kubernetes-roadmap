---
tags: [kubernetes, gitops, argocd, flux, phase/4]
phase: 4
topic: GitOps
status: not-started
created: 2026-03-01
---

# GitOps — ArgoCD & Flux

GitOps is a deployment model where **Git is the single source of truth** for cluster state. Changes are made via PRs, and a controller continuously reconciles the cluster to match.

## GitOps Principles (OpenGitOps)

1. **Declarative** — desired state expressed in declarative files
2. **Versioned and immutable** — stored in Git, full history
3. **Pulled automatically** — agent pulls from Git, not CI pushes
4. **Continuously reconciled** — agent detects and corrects drift

## ArgoCD

### Installation & Setup
```bash
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get initial admin password
argocd admin initial-password -n argocd

# Login
kubectl port-forward svc/argocd-server -n argocd 8080:443
argocd login localhost:8080
```

### App of Apps Pattern

One ArgoCD Application manages all other Applications — a single entry point for the entire cluster's desired state.

```yaml
# apps/app-of-apps.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app-of-apps
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/myorg/cluster-config
    path: apps/
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

```yaml
# apps/web-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/myorg/web-app
    path: k8s/overlays/production
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### Multi-Tenant ArgoCD

```yaml
# AppProject — limit what an Application can deploy
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-a
  namespace: argocd
spec:
  sourceRepos:
    - https://github.com/myorg/team-a-*
  destinations:
    - namespace: team-a-*
      server: https://kubernetes.default.svc
  clusterResourceWhitelist: []       # no cluster-scoped resources
  namespaceResourceBlacklist:
    - group: ""
      kind: ResourceQuota            # team can't change their own quotas
```

### ArgoCD Image Updater

Automatically update image tags in Git when new images are pushed:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

```yaml
# Annotation on Application
annotations:
  argocd-image-updater.argoproj.io/image-list: web=ghcr.io/myorg/web-app
  argocd-image-updater.argoproj.io/web.update-strategy: semver
  argocd-image-updater.argoproj.io/web.allow-tags: regexp:^v[0-9]+\.[0-9]+\.[0-9]+$
```

## Flux

Flux is a GitOps toolkit — modular components rather than a monolithic app.

```bash
flux install

# Bootstrap with GitHub
flux bootstrap github \
  --owner=myorg \
  --repository=cluster-config \
  --branch=main \
  --path=clusters/production
```

```yaml
# GitRepository — source of truth
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: cluster-config
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/myorg/cluster-config
  ref:
    branch: main
---
# Kustomization — what to apply
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: cluster-config
  path: ./apps/production
  prune: true
  healthChecks:
    - apiVersion: apps/v1
      kind: Deployment
      name: web-app
      namespace: production
```

### Flux Image Automation

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: web-app
  namespace: flux-system
spec:
  image: ghcr.io/myorg/web-app
  interval: 5m
---
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: web-app
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: web-app
  policy:
    semver:
      range: '>=1.0.0'
```

## ArgoCD vs Flux

| Feature | ArgoCD | Flux |
|---------|--------|------|
| Architecture | Monolithic app + UI | Modular controllers |
| UI | Built-in | None (use Weave GitOps) |
| Multi-cluster | Yes (agent model) | Yes (remote kubeconfig) |
| Image updates | Image Updater (separate) | Built-in |
| Learning curve | Gentler | Steeper |
| CNCF status | Graduated | Graduated |

## Repository Structure for GitOps

```
cluster-config/
├── clusters/
│   ├── production/
│   │   ├── flux-system/        # Flux bootstrap files
│   │   └── apps.yaml           # points to apps/production
│   └── staging/
├── apps/
│   ├── base/                   # shared manifests
│   ├── production/             # production overlays
│   └── staging/                # staging overlays
└── infrastructure/
    ├── cert-manager/
    ├── ingress-nginx/
    └── monitoring/
```

## Resources

| Resource | Link |
|----------|------|
| ArgoCD docs | https://argo-cd.readthedocs.io |
| Flux docs | https://fluxcd.io/docs |
| GitOps Cookbook (O'Reilly) | Book |
| OpenGitOps principles | https://opengitops.dev |

## Practice

- [ ] Deploy ArgoCD and manage an app entirely through Git (no kubectl apply manually)
- [ ] Implement the app-of-apps pattern for a multi-service application
- [ ] Set up ArgoCD Image Updater to automatically deploy new tags
- [ ] Force a drift (manually change a deployment) and watch ArgoCD self-heal
- [ ] Bootstrap Flux on a cluster and manage all infrastructure through Git

---

← [[index|Phase 4 Index]] | Next: [[Security]] →
