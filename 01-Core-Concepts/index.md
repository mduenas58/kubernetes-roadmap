---
tags: [kubernetes, core-concepts, phase/1]
phase: 1
status: not-started
created: 2026-03-01
---

# Phase 1 — Core Concepts

The heart of Kubernetes. Everything else builds on these fundamentals.

## Topics

| Topic | Note | Status |
|-------|------|--------|
| Architecture | [[Architecture]] | ⬜ |
| Pods | [[Pods]] | ⬜ |
| Workloads | [[Workloads]] | ⬜ |
| Networking (Services & Ingress) | [[Networking]] | ⬜ |
| Configuration (ConfigMaps & Secrets) | [[Configuration]] | ⬜ |
| kubectl | [[kubectl]] | ⬜ |

## Local Cluster Setup

Before starting this phase, set up a local cluster. Pick one:

| Tool | Best For | Install |
|------|----------|---------|
| **minikube** | Beginners, single-node | `brew install minikube` |
| **kind** | CI, multi-node on laptop | `brew install kind` |
| **k3d** | Fast, lightweight k3s in Docker | `brew install k3d` |

```bash
# Quickstart with minikube
minikube start
kubectl get nodes
```

## Time Estimate

- 3–4 weeks at 2 hours/day
- 2 weeks at 4 hours/day

## Phase Exit Checklist

- [ ] Can explain the Kubernetes control plane components from memory
- [ ] Can deploy an app with a Deployment and scale it
- [ ] Can expose an app with a Service and Ingress
- [ ] Can inject config with ConfigMaps and Secrets
- [ ] Can use `kubectl` fluently: get, describe, logs, exec, apply, delete
- [ ] Understand rolling updates and rollbacks

---

← [[../00-Prerequisites/index|Phase 0 – Prerequisites]] | Next: [[../02-Intermediate/index|Phase 2 – Intermediate]] →
