---
tags: [kubernetes, labs, hands-on]
created: 2026-03-01
---

# Labs — Hands-On Practice

Labs are organized by phase. Each lab builds on the previous one. Do every lab — reading alone is not enough to learn Kubernetes.

## Local Cluster Setup (Do This First)

```bash
# Option A: minikube (simplest)
brew install minikube
minikube start --cpus=4 --memory=8192
minikube addons enable metrics-server
minikube addons enable ingress

# Option B: k3d (faster, multi-node)
brew install k3d
k3d cluster create dev --agents 2 --port "80:80@loadbalancer" --port "443:443@loadbalancer"

# Option C: kind
brew install kind
kind create cluster --config kind-config.yaml
```

---

## Phase 0 Labs — Prerequisites

| Lab | Goal |
|-----|------|
| **Lab 00-1** | Write and build a Dockerfile for a Python web app |
| **Lab 00-2** | Run the container, expose port 8080, test with curl |
| **Lab 00-3** | Multi-stage build — reduce image from 800MB to <100MB |
| **Lab 00-4** | Compose a multi-service app (app + Redis) with Docker Compose |
| **Lab 00-5** | Write a YAML file that covers all data types — lint with yamllint |

---

## Phase 1 Labs — Core Concepts

| Lab | Goal |
|-----|------|
| **Lab 01-1** | Deploy an nginx Pod manually — inspect with describe, logs, exec |
| **Lab 01-2** | Write a multi-container Pod (app + log-sidecar sharing a volume) |
| **Lab 01-3** | Deploy an app with a Deployment (3 replicas) — scale up to 5, back to 2 |
| **Lab 01-4** | Trigger a rolling update — watch pods cycle — roll back |
| **Lab 01-5** | Deploy a StatefulSet — delete a pod — verify stable hostname |
| **Lab 01-6** | Expose an app with ClusterIP, NodePort, and LoadBalancer — test each |
| **Lab 01-7** | Set up Ingress with two path-based routes to two services |
| **Lab 01-8** | Inject config with a ConfigMap (env + volume) |
| **Lab 01-9** | Store a password in a Secret — use it in a pod, verify it doesn't show in logs |
| **Lab 01-10** | Create two namespaces — verify resource isolation |

---

## Phase 2 Labs — Intermediate

| Lab | Goal |
|-----|------|
| **Lab 02-1** | Deploy PostgreSQL with a PVC — write data — delete pod — verify persistence |
| **Lab 02-2** | Use a StorageClass for dynamic PVC provisioning |
| **Lab 02-3** | Create a Role that allows read-only pod access in one namespace |
| **Lab 02-4** | Bind the Role to a ServiceAccount — verify with `kubectl auth can-i` |
| **Lab 02-5** | Apply default-deny Network Policy to a namespace — verify traffic is blocked |
| **Lab 02-6** | Add a policy allowing only the frontend Service to reach the backend |
| **Lab 02-7** | Set resource requests/limits — watch OOMKill with an undersized limit |
| **Lab 02-8** | Configure HPA — generate CPU load — watch pods scale up and down |
| **Lab 02-9** | Install PostgreSQL with Helm — override values — upgrade — rollback |
| **Lab 02-10** | Write your own Helm chart for a simple app with staging/prod values |

---

## Phase 3 Labs — Advanced

| Lab | Goal |
|-----|------|
| **Lab 03-1** | Bootstrap a 3-node cluster with kubeadm (VMs or cloud) |
| **Lab 03-2** | Perform a Kubernetes cluster upgrade (n → n+1) |
| **Lab 03-3** | Install kube-prometheus-stack — build a custom Grafana dashboard |
| **Lab 03-4** | Write a PrometheusRule that fires when a pod crashes more than 3 times |
| **Lab 03-5** | Set up a GitHub Actions pipeline that builds, pushes, and deploys |
| **Lab 03-6** | Set up ArgoCD — deploy app from Git — trigger self-heal |
| **Lab 03-7** | Install Istio — enable mTLS — configure a 90/10 canary deployment |
| **Lab 03-8** | Write a CRD + controller that creates a ConfigMap for each custom resource |
| **Lab 03-9** | Install cert-manager — issue a Let's Encrypt cert for an Ingress |
| **Lab 03-10** | Set up Loki + Promtail — query container logs from Grafana |

---

## Phase 4 Labs — Master

| Lab | Goal |
|-----|------|
| **Lab 04-1** | Implement full GitOps: all cluster changes via PR → ArgoCD |
| **Lab 04-2** | App-of-apps pattern — one ArgoCD App to rule them all |
| **Lab 04-3** | Run kube-bench — fix at least 5 CIS benchmark findings |
| **Lab 04-4** | Apply Pod Security Standards (restricted) to a namespace — fix violations |
| **Lab 04-5** | Write an OPA Gatekeeper policy — test enforcement |
| **Lab 04-6** | Install Falco — trigger a rule by exec-ing into a pod |
| **Lab 04-7** | Set up etcd encryption at rest — verify secrets are encrypted in etcd |
| **Lab 04-8** | Provision a cluster with Cluster API (Docker provider) |
| **Lab 04-9** | Deploy an app to two clusters with ArgoCD ApplicationSet |
| **Lab 04-10** | Contribute a documentation fix to `kubernetes/website` |

---

## Useful Cluster Images for Labs

```bash
# Network debugging
kubectl run netshoot --image=nicolaka/netshoot -it --rm -- bash

# Curl from inside cluster
kubectl run curl --image=curlimages/curl -it --rm -- sh

# Generic debug
kubectl run debug --image=busybox -it --rm -- sh

# DNS testing
kubectl run dnsutils --image=gcr.io/kubernetes-e2e-test-images/dnsutils -it --rm -- bash
```
