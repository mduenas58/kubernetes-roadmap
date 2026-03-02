---
tags: [kubernetes, configmap, secrets, configuration, phase/1]
phase: 1
topic: Configuration
status: not-started
created: 2026-03-01
---

# Configuration — ConfigMaps & Secrets

Keep configuration out of container images. Kubernetes provides two objects for this: ConfigMaps for non-sensitive data, Secrets for sensitive data.

## ConfigMap

Stores arbitrary **non-sensitive** key-value pairs or files.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"
  config.yaml: |
    server:
      port: 8080
      timeout: 30s
```

### Using a ConfigMap

**As environment variables:**
```yaml
spec:
  containers:
    - name: app
      envFrom:
        - configMapRef:
            name: app-config       # all keys become env vars
      env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL       # single key
```

**As a mounted file:**
```yaml
spec:
  containers:
    - name: app
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config           # config.yaml appears at /etc/config/config.yaml
```

> [!TIP]
> Mounted ConfigMaps are **updated automatically** when the ConfigMap changes (with a short delay). Environment variables are NOT updated — the pod must restart.

## Secret

Stores **sensitive** data: passwords, tokens, certificates, API keys.

> [!WARNING]
> Kubernetes Secrets are base64-encoded, not encrypted, by default. They are stored in etcd in plaintext unless etcd encryption is configured. For real security, use Vault, Sealed Secrets, or External Secrets Operator.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=           # echo -n "admin" | base64
  password: c3VwZXJzZWNyZXQ=  # echo -n "supersecret" | base64
```

Or use `stringData` (plain text — Kubernetes encodes it):
```yaml
stringData:
  username: admin
  password: supersecret
```

### Secret Types

| Type | Use |
|------|-----|
| `Opaque` | Arbitrary data (default) |
| `kubernetes.io/dockerconfigjson` | Image pull secrets |
| `kubernetes.io/tls` | TLS certificates |
| `kubernetes.io/service-account-token` | Service account tokens |

### Using a Secret (same as ConfigMap)
```yaml
envFrom:
  - secretRef:
      name: db-credentials

# Or as volume (files not readable in env logs)
volumes:
  - name: creds
    secret:
      secretName: db-credentials
```

## Namespaces

Namespaces provide **virtual cluster isolation** within a cluster.

```bash
# Create namespace
kubectl create namespace staging

# Work in a namespace
kubectl get pods -n staging
kubectl apply -f app.yaml -n staging

# Set default namespace for current context
kubectl config set-context --current --namespace=staging
```

Use namespaces to separate:
- Environments (dev / staging / prod) on a shared cluster
- Teams or applications
- System components (`kube-system`) from user workloads

**Resources that are NOT namespaced:** Nodes, PersistentVolumes, ClusterRoles, StorageClasses.

## Resources

| Resource | Link |
|----------|------|
| ConfigMaps | https://kubernetes.io/docs/concepts/configuration/configmap |
| Secrets | https://kubernetes.io/docs/concepts/configuration/secret |
| Namespaces | https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces |

## Practice

- [ ] Create a ConfigMap and inject it as both env vars and a mounted file
- [ ] Update a mounted ConfigMap and observe the file change without a pod restart
- [ ] Create a Secret for database credentials — mount it as a file, not an env var
- [ ] Create two namespaces and verify resources in one are not visible in the other
- [ ] Create a docker-registry Secret and use it as an imagePullSecret

---

← [[Networking]] | Next: [[kubectl]] →
