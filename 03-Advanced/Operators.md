---
tags: [kubernetes, operators, crd, controller, phase/3]
phase: 3
topic: Operators & CRDs
status: not-started
created: 2026-03-01
---

# Operators & Custom Resource Definitions

Operators extend Kubernetes to manage complex stateful applications (databases, queues, ML systems) using the same reconciliation pattern as built-in controllers.

## Custom Resource Definitions (CRDs)

A CRD teaches Kubernetes about a new resource type.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine:
                  type: string
                  enum: [postgres, mysql]
                version:
                  type: string
                replicas:
                  type: integer
                  minimum: 1
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
    shortNames: [db]
```

Once applied, you can create custom resources:
```yaml
apiVersion: example.com/v1
kind: Database
metadata:
  name: my-db
spec:
  engine: postgres
  version: "15"
  replicas: 3
```

## The Operator Pattern

An operator = **CRD** + **controller** that watches the CRD and reconciles state.

```
Desired state (CR spec)  ──► Controller ──► Actual state (Pods, PVCs, etc.)
                              reconciles
                              continuously
```

The controller loop:
1. Watch for changes to the Custom Resource
2. Compare desired state (spec) to actual state
3. Take action to converge (create/update/delete Kubernetes resources)
4. Update the CR status field with current state

## Building an Operator with controller-runtime (Go)

```bash
# Scaffold with Operator SDK
operator-sdk init --domain example.com --repo github.com/me/db-operator
operator-sdk create api --group example --version v1 --kind Database --resource --controller
```

```go
// controllers/database_controller.go
func (r *DatabaseReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := log.FromContext(ctx)

    // 1. Fetch the Database instance
    db := &examplev1.Database{}
    if err := r.Get(ctx, req.NamespacedName, db); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // 2. Check if StatefulSet already exists, if not create it
    found := &appsv1.StatefulSet{}
    err := r.Get(ctx, types.NamespacedName{Name: db.Name, Namespace: db.Namespace}, found)
    if errors.IsNotFound(err) {
        sts := r.statefulSetForDB(db)
        log.Info("Creating StatefulSet", "name", sts.Name)
        return ctrl.Result{}, r.Create(ctx, sts)
    }

    // 3. Reconcile replicas
    if *found.Spec.Replicas != db.Spec.Replicas {
        found.Spec.Replicas = &db.Spec.Replicas
        return ctrl.Result{}, r.Update(ctx, found)
    }

    return ctrl.Result{}, nil
}
```

## Operator SDK Tools

```bash
# Go-based operator
operator-sdk init --plugins go/v4

# Helm-based operator (wrap a Helm chart as an operator)
operator-sdk init --plugins helm/v1
operator-sdk create api --helm-chart=nginx

# Ansible-based operator
operator-sdk init --plugins ansible/v1
```

## Well-Known Operators

| Operator | Manages |
|----------|---------|
| **Prometheus Operator** | Prometheus, Alertmanager, ServiceMonitors |
| **cert-manager** | TLS certificates from Let's Encrypt/Vault |
| **Strimzi** | Apache Kafka clusters |
| **CloudNativePG** | PostgreSQL clusters |
| **Argo CD** | GitOps deployments |
| **Rook** | Ceph storage clusters |
| **Cluster API** | Kubernetes cluster provisioning |

## Admission Webhooks

Webhooks extend the API server with custom validation and mutation logic.

```
kubectl apply → API Server → Admission Webhook → etcd
                              ↑
                      ValidatingWebhook: reject if invalid
                      MutatingWebhook: modify before storing
```

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: validate-resource-limits
webhooks:
  - name: validate.limits.example.com
    rules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE", "UPDATE"]
        resources: ["pods"]
    clientConfig:
      service:
        name: webhook-service
        namespace: webhook-system
        path: "/validate-pods"
      caBundle: <base64-ca-cert>
    failurePolicy: Fail
```

## Resources

| Resource | Link |
|----------|------|
| Operator SDK | https://sdk.operatorframework.io |
| controller-runtime | https://github.com/kubernetes-sigs/controller-runtime |
| OperatorHub | https://operatorhub.io |
| Programming Kubernetes (O'Reilly) | Book |

## Practice

- [ ] Install cert-manager and issue a TLS certificate for a Service
- [ ] Write a minimal CRD and create a custom resource with it
- [ ] Scaffold a Go operator that watches a CRD and creates a ConfigMap
- [ ] Study the Prometheus Operator source — read how ServiceMonitors work
- [ ] Write a MutatingWebhook that injects an annotation into all Pods

---

← [[Service-Mesh]] | Next Phase: [[../04-Master/index|Phase 4 – Master]] →
