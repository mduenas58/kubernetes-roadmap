---
tags: [kubernetes, storage, pv, pvc, storageclass, csi, phase/2]
phase: 2
topic: Storage
status: not-started
created: 2026-03-01
---

# Storage — PV, PVC, StorageClass

Kubernetes separates **what storage is needed** (PVC) from **how storage is provided** (PV/StorageClass). This lets developers request storage without knowing the underlying infrastructure.

## Storage Hierarchy

```
StorageClass  ──► defines HOW to provision (which driver, parameters)
      │
      ▼
PersistentVolume (PV)  ──► actual storage resource (can be pre-provisioned or dynamic)
      │ bound to
      ▼
PersistentVolumeClaim (PVC)  ──► developer's request for storage
      │ mounted into
      ▼
Pod  ──► uses the storage via a volume mount
```

## PersistentVolume (PV)

A cluster-level resource representing a piece of storage. Created by an admin (static) or automatically by a StorageClass (dynamic).

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce       # RWO: one node at a time
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/my-pv     # local path (dev only — use a real driver in prod)
```

## PersistentVolumeClaim (PVC)

A namespaced request for storage. Bound to a matching PV.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-ssd   # must match StorageClass name
```

**Using the PVC in a Pod:**
```yaml
spec:
  containers:
    - name: db
      image: postgres:15
      volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: db-storage
  volumes:
    - name: db-storage
      persistentVolumeClaim:
        claimName: db-pvc
```

## StorageClass

Defines a **template for dynamic provisioning**. When a PVC references a StorageClass, the provisioner automatically creates a PV.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs     # or ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
reclaimPolicy: Delete                  # Delete or Retain
volumeBindingMode: WaitForFirstConsumer
```

Mark a StorageClass as default:
```bash
kubectl patch storageclass fast-ssd -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Access Modes

| Mode | Short | Meaning |
|------|-------|---------|
| `ReadWriteOnce` | RWO | One node can read+write |
| `ReadOnlyMany` | ROX | Many nodes can read |
| `ReadWriteMany` | RWX | Many nodes can read+write |
| `ReadWriteOncePod` | RWOP | One pod can read+write |

> [!NOTE]
> `ReadWriteMany` requires a distributed filesystem (NFS, CephFS, EFS). Most block storage (EBS, GCE PD) only supports `ReadWriteOnce`.

## Reclaim Policy

| Policy | What happens when PVC is deleted |
|--------|----------------------------------|
| `Retain` | PV stays, must be manually cleaned and reused |
| `Delete` | PV and underlying storage are deleted |
| `Recycle` | Deprecated — basic scrub and reuse |

## CSI — Container Storage Interface

Modern Kubernetes uses **CSI drivers** instead of in-tree plugins. Each cloud provider and storage vendor publishes a CSI driver:
- `ebs.csi.aws.com` — AWS EBS
- `pd.csi.storage.gke.io` — GCP Persistent Disk
- `disk.csi.azure.com` — Azure Disk
- `rook-ceph.rbd.csi.ceph.com` — Ceph (on-prem)

## Ephemeral Volumes (Non-Persistent)

| Type | Use |
|------|-----|
| `emptyDir` | Temporary dir, shared between containers, deleted with Pod |
| `hostPath` | Mounts host node filesystem (dev only, security risk) |
| `configMap` / `secret` | Inject config/secrets as files |
| `projected` | Combine multiple sources into one volume |

## Resources

| Resource | Link |
|----------|------|
| Storage concepts | https://kubernetes.io/docs/concepts/storage |
| CSI drivers list | https://kubernetes-csi.github.io/docs/drivers.html |

## Practice

- [ ] Deploy PostgreSQL with a PVC — delete and recreate the pod, verify data persists
- [ ] Create a StorageClass for dynamic provisioning (local-path-provisioner on k3d)
- [ ] Use a StatefulSet with `volumeClaimTemplates` — verify each pod gets its own PVC
- [ ] Inspect PV binding lifecycle: `Pending → Bound → Released`
- [ ] Test what happens with `Retain` policy when PVC is deleted

---

← [[index|Phase 2 Index]] | Next: [[RBAC]] →
