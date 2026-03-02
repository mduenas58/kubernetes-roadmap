---
tags: [kubernetes, cicd, github-actions, tekton, gitops, phase/3]
phase: 3
topic: CI/CD
status: not-started
created: 2026-03-01
---

# CI/CD for Kubernetes

A reliable delivery pipeline means no manual `kubectl apply`. Changes are triggered by Git commits, tested automatically, and deployed consistently.

## Delivery Models

```
Push-based CI/CD               Pull-based (GitOps)
─────────────────────          ──────────────────────────
1. Code pushed to Git          1. Code pushed to Git
2. CI builds image             2. CI builds image, updates manifests
3. CI runs kubectl apply       3. GitOps agent (ArgoCD/Flux) detects diff
4. Changes applied directly    4. Agent applies changes from inside cluster
```

> [!NOTE]
> Pull-based GitOps is the more secure approach — no external system needs cluster credentials.

## GitHub Actions — Push-Based

```yaml
# .github/workflows/deploy.yaml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build and push image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Configure kubectl
        uses: azure/setup-kubectl@v3

      - name: Set kubeconfig
        run: echo "${{ secrets.KUBECONFIG }}" | base64 -d > ~/.kube/config

      - name: Update deployment image
        run: |
          kubectl set image deployment/web-app \
            web=ghcr.io/${{ github.repository }}:${{ github.sha }} \
            -n production

      - name: Verify rollout
        run: kubectl rollout status deployment/web-app -n production
```

## GitHub Actions with Helm

```yaml
      - name: Deploy with Helm
        run: |
          helm upgrade --install web-app ./charts/web-app \
            --namespace production \
            --create-namespace \
            --set image.tag=${{ github.sha }} \
            --wait \
            --timeout 5m
```

## Kustomize — Environment Overlays

Kustomize lets you maintain base manifests and environment-specific overlays without templating.

```
k8s/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── staging/
    │   ├── replica-patch.yaml    # replicas: 1
    │   └── kustomization.yaml
    └── production/
        ├── replica-patch.yaml    # replicas: 5
        └── kustomization.yaml
```

```yaml
# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
images:
  - name: my-app
    newTag: "abc1234"            # CI updates this line
patches:
  - path: replica-patch.yaml
```

```bash
kubectl apply -k k8s/overlays/production/
```

## Tekton — Kubernetes-Native CI/CD

Tekton runs CI/CD pipelines as Kubernetes objects.

```yaml
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: build-and-deploy
spec:
  tasks:
    - name: build
      taskRef:
        name: buildah
    - name: deploy
      runAfter: [build]
      taskRef:
        name: kubernetes-deploy
```

## ArgoCD — GitOps (Introduction)

> Full coverage in [[../04-Master/GitOps|GitOps (Phase 4)]]

Quick start:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

```yaml
# Application manifest
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/myorg/myrepo
    path: k8s/overlays/production
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Image Update Strategy

| Strategy | Tool | How |
|----------|------|-----|
| `latest` tag (not recommended) | — | Always pulls latest |
| SHA tag | CI/CD | Pin exact image digest |
| Semantic versioning | ArgoCD Image Updater | Auto-bump on new semver tag |
| Immutable tags | Registry policy | Enforce via OPA/Admission |

## Secrets in CI/CD

Never store raw secrets in Git. Options:
- **GitHub Actions Secrets** — injected as environment variables at runtime
- **Sealed Secrets** — encrypt secrets, store ciphertext in Git
- **External Secrets Operator** — fetch from Vault/AWS SSM at deploy time
- **SOPS + age** — encrypt files in Git

## Resources

| Resource | Link |
|----------|------|
| GitHub Actions | https://docs.github.com/en/actions |
| Kustomize | https://kustomize.io |
| Tekton | https://tekton.dev |
| ArgoCD | https://argo-cd.readthedocs.io |

## Practice

- [ ] Build a GitHub Actions workflow that builds, pushes, and deploys an image
- [ ] Set up staging and production overlays with Kustomize
- [ ] Install ArgoCD and deploy an app from a Git repo
- [ ] Trigger a bad deployment and watch ArgoCD self-heal
- [ ] Set up Sealed Secrets for safe secret management in Git

---

← [[Observability]] | Next: [[Service-Mesh]] →
