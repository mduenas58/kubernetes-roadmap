---
tags: [kubernetes, security, cks, opa, falco, hardening, phase/4]
phase: 4
topic: Security
status: not-started
created: 2026-03-01
---

# Security Hardening

Kubernetes has a large attack surface. Hardening it requires securing the cluster at every layer.

## Security Layers

```
1. Infrastructure   ── node OS, cloud IAM, network
2. Cluster          ── API server, etcd, kubeconfig
3. Container        ── image scanning, runtime security
4. Application      ── RBAC, Network Policies, Secrets management
5. Supply Chain     ── image signing, SBOM, provenance
```

## CIS Kubernetes Benchmark

The Center for Internet Security publishes hardening recommendations. Run the automated check:

```bash
# kube-bench — checks against CIS benchmark
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
kubectl logs job/kube-bench
```

Key findings it checks:
- API server flags (anonymous auth disabled, audit logging on, etc.)
- etcd TLS and access control
- kubelet hardening
- RBAC least privilege
- Network Policies

## API Server Hardening

```yaml
# kube-apiserver flags
--anonymous-auth=false
--audit-log-path=/var/log/kubernetes/audit.log
--audit-log-maxage=30
--audit-policy-file=/etc/kubernetes/audit-policy.yaml
--enable-admission-plugins=NodeRestriction,PodSecurity
--tls-min-version=VersionTLS12
```

### Audit Policy

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  - level: None           # don't log read-only URLs
    verbs: ["get", "watch", "list"]
    resources:
      - resources: ["endpoints", "services", "pods"]
  - level: Metadata       # log all requests to secrets (metadata only)
    resources:
      - resources: ["secrets", "configmaps"]
  - level: RequestResponse # log writes fully
    verbs: ["create", "update", "patch", "delete"]
```

## Pod Security Standards

Replacement for PodSecurityPolicy (removed in 1.25). Enforced via the built-in `PodSecurity` admission controller.

Three levels:
| Level | What it blocks |
|-------|---------------|
| `privileged` | No restrictions |
| `baseline` | Prevents most known escalations |
| `restricted` | Follows security best practices |

```yaml
# Enforce restricted policy on a namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

Compliant Pod:
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop: [ALL]
        readOnlyRootFilesystem: true
```

## OPA Gatekeeper — Policy as Code

Enforce custom policies cluster-wide using Open Policy Agent.

```bash
helm install gatekeeper opa/gatekeeper \
  --namespace gatekeeper-system \
  --create-namespace
```

```yaml
# ConstraintTemplate — defines a policy template
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: requirelabels
spec:
  crd:
    spec:
      names:
        kind: RequireLabels
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package requirelabels
        violation[{"msg": msg}] {
          required := input.parameters.labels[_]
          not input.review.object.metadata.labels[required]
          msg := sprintf("Missing label: %v", [required])
        }
---
# Constraint — applies the template
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: RequireLabels
metadata:
  name: require-app-label
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    labels: ["app", "version", "owner"]
```

## Falco — Runtime Security

Detects anomalous behavior at runtime: unexpected shell spawns, network connections, file reads.

```bash
helm install falco falcosecurity/falco \
  --namespace falco \
  --create-namespace \
  --set driver.kind=ebpf
```

Example rule:
```yaml
- rule: Terminal Shell in Container
  desc: A shell was spawned in a container
  condition: >
    evt.type = execve and
    container and
    proc.name in (shell_binaries)
  output: >
    Shell spawned in container
    (user=%user.name container=%container.name image=%container.image.repository)
  priority: WARNING
```

## Image Security

```bash
# Scan images with Trivy
trivy image nginx:latest
trivy image --severity CRITICAL,HIGH myapp:1.0

# Sign images with Cosign
cosign sign --key cosign.key ghcr.io/myorg/myapp:1.0

# Verify signature
cosign verify --key cosign.pub ghcr.io/myorg/myapp:1.0
```

## etcd Encryption at Rest

```yaml
# /etc/kubernetes/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources: [secrets]
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <base64-encoded-32-byte-key>
      - identity: {}   # fallback for unencrypted data
```

```bash
# Apply to API server
--encryption-provider-config=/etc/kubernetes/encryption-config.yaml

# Re-encrypt existing secrets
kubectl get secrets -A -o json | kubectl replace -f -
```

## Secrets Management (External)

| Tool | When to use |
|------|------------|
| **Vault** (HashiCorp) | Enterprise, dynamic secrets, PKI |
| **External Secrets Operator** | Sync from Vault/AWS/GCP to k8s Secrets |
| **Sealed Secrets** | Encrypt secrets for Git storage |
| **SOPS + age** | Simpler Git-encrypted secrets |

## CKS Exam Topics

- [ ] Cluster hardening (RBAC, API server, network policies)
- [ ] System hardening (seccomp, AppArmor, syscall restrictions)
- [ ] Minimize microservice vulnerabilities (pod security, OPA)
- [ ] Supply chain security (image scanning, signing, admission)
- [ ] Runtime security (Falco, immutable containers)
- [ ] Audit logging
- [ ] etcd encryption

## Resources

| Resource | Link |
|----------|------|
| CIS Kubernetes Benchmark | https://www.cisecurity.org/benchmark/kubernetes |
| kube-bench | https://github.com/aquasecurity/kube-bench |
| OPA Gatekeeper | https://open-policy-agent.github.io/gatekeeper |
| Falco | https://falco.org |
| Trivy | https://aquasecurity.github.io/trivy |

---

← [[GitOps]] | Next: [[Multi-Cluster]] →
