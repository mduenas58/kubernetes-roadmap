---
tags: [kubernetes, workloads, deployments, statefulsets, daemonsets, jobs, phase/1]
phase: 1
topic: Workloads
status: not-started
created: 2026-03-01
---

# Workloads

Pods are rarely created directly. Workload resources manage Pods for you — handling replication, updates, and scheduling.

## Deployment

The most common workload. Use for **stateless** applications.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web
          image: nginx:1.25
          ports:
            - containerPort: 80
```

**Key operations:**
```bash
kubectl rollout status deployment/web-app
kubectl rollout history deployment/web-app
kubectl rollout undo deployment/web-app          # rollback
kubectl rollout undo deployment/web-app --to-revision=2
kubectl scale deployment web-app --replicas=5
```

## ReplicaSet

Ensures N identical pods are running. **Deployments manage ReplicaSets** — you rarely create ReplicaSets directly.

- Deployment → creates ReplicaSet → creates Pods
- A new ReplicaSet is created on each Deployment update; old ones are kept for rollback history

## StatefulSet

Use for **stateful** applications: databases, queues, distributed systems.

Differences from Deployment:
| Feature | Deployment | StatefulSet |
|---------|------------|-------------|
| Pod names | Random (`web-abc12`) | Ordered (`db-0`, `db-1`) |
| Pod identity | Interchangeable | Stable, permanent |
| Storage | Shared or ephemeral | Dedicated PVC per pod |
| Startup order | Parallel | Sequential (0 → 1 → 2) |
| DNS | Single Service | Per-pod DNS (`db-0.db-svc`) |

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: "db-svc"
  replicas: 3
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: postgres
          image: postgres:15
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

## DaemonSet

Ensures **one pod runs on every node** (or a subset). Used for:
- Node monitoring agents (Prometheus Node Exporter, Datadog agent)
- Log collectors (Fluentd, Filebeat)
- Network plugins (CNI agents, kube-proxy itself)
- Storage daemons

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      containers:
        - name: fluentd
          image: fluentd:latest
```

## Job

Runs a Pod to **completion**. Use for one-off tasks: batch processing, migrations, report generation.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 3
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: myapp:latest
          command: ["python", "manage.py", "migrate"]
```

## CronJob

Runs a Job on a **cron schedule**.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup
spec:
  schedule: "0 2 * * *"      # 2am every day
  concurrencyPolicy: Forbid  # don't run if previous still running
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: cleanup
              image: myapp:latest
              command: ["python", "cleanup.py"]
```

## Choosing the Right Workload

```
Stateless app?         → Deployment
Stateful app/DB?       → StatefulSet
Run on every node?     → DaemonSet
One-time task?         → Job
Scheduled task?        → CronJob
```

## Resources

| Resource | Link |
|----------|------|
| Workloads overview | https://kubernetes.io/docs/concepts/workloads |
| Deployment strategies | https://kubernetes.io/docs/concepts/workloads/controllers/deployment |

## Practice

- [ ] Deploy an app with 3 replicas, trigger a rolling update, then roll back
- [ ] Deploy a StatefulSet with persistent storage — delete a pod and verify it comes back with the same data
- [ ] Deploy a DaemonSet and verify a pod exists on every node
- [ ] Create a Job that runs a script and verify it completes
- [ ] Create a CronJob and manually trigger it with `kubectl create job --from`

---

← [[Pods]] | Next: [[Networking]] →
