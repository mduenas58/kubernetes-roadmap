---
tags: [kubernetes, helm, package-manager, phase/2]
phase: 2
topic: Helm
status: not-started
created: 2026-03-01
---

# Helm — The Kubernetes Package Manager

Helm lets you install, upgrade, and manage complex Kubernetes applications as versioned, configurable **charts**.

## Core Concepts

| Term | Meaning |
|------|---------|
| **Chart** | Package of Kubernetes manifests + templates |
| **Release** | A running instance of a chart in the cluster |
| **Repository** | Collection of charts (like npm registry) |
| **Values** | Configuration parameters for a chart |
| **Revision** | Version of a release (increments on each upgrade) |

## Installing Helm

```bash
brew install helm

# Verify
helm version
```

## Working with Repositories

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm search repo nginx
helm search repo bitnami/postgresql --versions
```

## Installing Charts

```bash
# Install with defaults
helm install my-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace

# Install with custom values
helm install my-postgres bitnami/postgresql \
  --namespace databases \
  --create-namespace \
  --set auth.postgresPassword=secret \
  --set primary.persistence.size=20Gi

# Install with values file
helm install my-postgres bitnami/postgresql \
  -f values-prod.yaml
```

## Managing Releases

```bash
helm list                          # list all releases
helm list -A                       # all namespaces
helm status my-nginx               # release status
helm get values my-postgres        # see applied values
helm get manifest my-postgres      # see rendered manifests

helm upgrade my-postgres bitnami/postgresql -f values-prod.yaml
helm rollback my-postgres 1        # rollback to revision 1
helm uninstall my-postgres
```

## Chart Structure

```
my-chart/
├── Chart.yaml          # Chart metadata (name, version, description)
├── values.yaml         # Default values
├── templates/          # Kubernetes manifests with Go templating
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # Template helpers (reusable snippets)
│   └── NOTES.txt       # Post-install instructions
└── charts/             # Subcharts (dependencies)
```

## Writing a Chart

```yaml
# Chart.yaml
apiVersion: v2
name: my-app
description: My application
type: application
version: 0.1.0          # chart version
appVersion: "1.2.3"     # app version
```

```yaml
# values.yaml
image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent

replicaCount: 2

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
```

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
```

## Templating Functions

```yaml
# String functions
{{ .Values.name | upper }}
{{ .Values.name | trunc 63 | trimSuffix "-" }}
{{ printf "%s-%s" .Release.Name .Chart.Name | quote }}

# Conditionals
{{- if .Values.ingress.enabled }}
  # ingress template
{{- end }}

# Loops
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}

# Default values
{{ .Values.service.port | default 80 }}
```

## Helm Secrets (Handling Sensitive Values)

Never commit plain secrets to values files. Options:
- **helm-secrets** plugin + SOPS/age encryption
- **External Secrets Operator** — fetch from Vault/AWS Secrets Manager
- CI/CD pipeline — inject values at deploy time, never stored in repo

```bash
helm plugin install https://github.com/jkroepke/helm-secrets
helm secrets install my-app . -f secrets.yaml.enc
```

## Resources

| Resource | Link |
|----------|------|
| Helm docs | https://helm.sh/docs |
| Artifact Hub (chart registry) | https://artifacthub.io |
| helm-secrets | https://github.com/jkroepke/helm-secrets |
| Helmfile (multiple charts) | https://github.com/helmfile/helmfile |

## Practice

- [ ] Install PostgreSQL and nginx-ingress with Helm
- [ ] Override values using `-f values.yaml` and `--set`
- [ ] Roll back a release and observe the revision history
- [ ] Create a Helm chart for one of your own apps
- [ ] Use a values file for staging and a different one for production

---

← [[Resource-Management]] | Next Phase: [[../03-Advanced/index|Phase 3 – Advanced]] →
