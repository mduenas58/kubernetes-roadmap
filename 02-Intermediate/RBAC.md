---
tags: [kubernetes, rbac, security, serviceaccount, phase/2]
phase: 2
topic: RBAC
status: not-started
created: 2026-03-01
---

# RBAC — Role-Based Access Control

Kubernetes RBAC controls **who** can do **what** to **which resources**.

## Core Objects

```
Subject (who)        + Role (what)         → Binding (grant)
─────────────────────────────────────────────────────────────
User / Group         + Role                → RoleBinding        (namespaced)
ServiceAccount       + ClusterRole         → ClusterRoleBinding (cluster-wide)
```

## Roles & ClusterRoles

A **Role** grants permissions within a single namespace.
A **ClusterRole** grants permissions cluster-wide (or can be bound in a namespace via RoleBinding).

```yaml
# Namespaced Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: staging
rules:
  - apiGroups: [""]           # "" = core API group
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "watch", "update", "patch"]
```

```yaml
# Cluster-wide ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
```

### Common Verbs

| Verb | HTTP Equivalent |
|------|----------------|
| `get` | GET (single) |
| `list` | GET (collection) |
| `watch` | GET + watch |
| `create` | POST |
| `update` | PUT |
| `patch` | PATCH |
| `delete` | DELETE |
| `deletecollection` | DELETE (collection) |

## RoleBindings & ClusterRoleBindings

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: staging
subjects:
  - kind: User
    name: alice
    apiGroup: rbac.authorization.k8s.io
  - kind: Group
    name: dev-team
    apiGroup: rbac.authorization.k8s.io
  - kind: ServiceAccount
    name: ci-bot
    namespace: staging
roleRef:
  kind: Role           # or ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## ServiceAccounts

Every pod runs as a ServiceAccount. By default it runs as `default` in its namespace — with minimal permissions.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-service
  namespace: production
```

Assign to a pod:
```yaml
spec:
  serviceAccountName: api-service
```

The ServiceAccount token is automatically mounted at:
`/var/run/secrets/kubernetes.io/serviceaccount/token`

Disable auto-mounting if the pod doesn't need API access:
```yaml
spec:
  automountServiceAccountToken: false
```

## Checking Permissions

```bash
# Can the current user do X?
kubectl auth can-i create pods
kubectl auth can-i delete deployments -n staging

# Can a specific user/SA do X?
kubectl auth can-i get secrets --as=alice
kubectl auth can-i list pods --as=system:serviceaccount:staging:ci-bot

# What can the current user do?
kubectl auth can-i --list
kubectl auth can-i --list -n staging
```

## Principle of Least Privilege

> [!WARNING]
> Never bind `cluster-admin` to application service accounts. Grant only the specific verbs and resources each identity actually needs.

Common mistakes:
- Giving a CI pipeline `cluster-admin` when it only needs to update Deployments
- Using the `default` ServiceAccount for apps that need API access
- Broad `*` resource/verb grants in Roles

## Resources

| Resource | Link |
|----------|------|
| RBAC docs | https://kubernetes.io/docs/reference/access-authn-authz/rbac |
| rbac-lookup tool | https://github.com/FairwindsOps/rbac-lookup |
| rakkess (access matrix) | https://github.com/corneliusweig/rakkess |

## Practice

- [ ] Create a Role that allows a CI bot to update Deployments but not read Secrets
- [ ] Bind it to a ServiceAccount and test with `kubectl auth can-i`
- [ ] Impersonate the service account: `kubectl --as=...` and verify it can and cannot do things
- [ ] Audit all ClusterRoleBindings with `cluster-admin` in your cluster
- [ ] Disable `automountServiceAccountToken` on Pods that don't need API access

---

← [[Storage]] | Next: [[Network-Policies]] →
