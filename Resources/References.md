---
tags: [kubernetes, resources, books, courses, certifications]
created: 2026-03-01
---

# Resources & References

## Books

| Title | Author | Best For |
|-------|--------|----------|
| **Kubernetes in Action** (2nd ed.) | Marko Lukša | Phase 1–2 deep understanding |
| **Programming Kubernetes** | Heptio | Operators, controllers, Phase 3–4 |
| **Kubernetes Patterns** | Ibryam & Huß | Design patterns for cloud-native apps |
| **Cloud Native DevOps with Kubernetes** | Arundel & Domingus | Practical ops, Phase 2–3 |
| **The Linux Command Line** | William Shotts | Phase 0 (free: linuxcommand.org) |

## Official Documentation

| Resource | URL |
|----------|-----|
| Kubernetes docs | https://kubernetes.io/docs |
| kubectl reference | https://kubernetes.io/docs/reference/kubectl |
| API reference | https://kubernetes.io/docs/reference/kubernetes-api |
| Helm docs | https://helm.sh/docs |
| CNCF landscape | https://landscape.cncf.io |

## Interactive Learning

| Resource | Type | Cost |
|----------|------|------|
| **Killercoda** | Browser labs | Free + paid |
| **KodeKloud** | Guided courses + labs | Paid |
| **A Cloud Guru** | Video + labs | Paid |
| **Play with Kubernetes** | Browser cluster | Free |
| **Instruqt** | Interactive scenarios | Free + paid |

## Video Courses

| Course | Platform | Best For |
|--------|----------|----------|
| **Kubernetes for the Absolute Beginners** (Mumshad) | Udemy / KodeKloud | Phase 0–1 |
| **Certified Kubernetes Administrator (CKA)** (Mumshad) | KodeKloud | Phase 1–2 + CKA prep |
| **TechWorld with Nana – Kubernetes Tutorials** | YouTube | Visual learners, free |
| **CKAD with Tests** (Mumshad) | KodeKloud | CKAD prep |

## Certification Preparation

### CKA — Certified Kubernetes Administrator
- **Exam format:** 2 hours, performance-based (live cluster tasks)
- **Key topics:** cluster setup, workloads, networking, storage, troubleshooting
- **Best prep:** KodeKloud CKA course + killer.sh mock exams

```bash
# Register at: https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/
# killer.sh — 2 free practice exams included with CKA purchase
```

### CKAD — Certified Kubernetes Application Developer
- **Exam format:** 2 hours, performance-based
- **Key topics:** app design, deployment, configuration, observability, services
- **Best prep:** KodeKloud CKAD course + killer.sh

```bash
# Register at: https://training.linuxfoundation.org/certification/certified-kubernetes-application-developer-ckad/
```

### CKS — Certified Kubernetes Security Specialist
- **Requires:** active CKA
- **Exam format:** 2 hours, performance-based
- **Key topics:** cluster hardening, supply chain, runtime security, microservice security
- **Best prep:** KodeKloud CKS course + killer.sh

```bash
# Register at: https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist-cks/
```

## CKA/CKAD Exam Tips

> [!TIP]
> The exam is hands-on, timed, and stressful. Practice with kubectl until the commands are muscle memory.

```bash
# Must-know shortcuts
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"

# Generate manifests fast (don't type from scratch)
k create deploy web --image=nginx $do > deploy.yaml
k run pod --image=nginx $do > pod.yaml
k create svc clusterip web --tcp=80 $do > svc.yaml

# Set context at start of each question
kubectl config use-context <cluster-name>

# Set namespace
kubectl config set-context --current --namespace=<namespace>
```

## Community & Conferences

| Resource | URL |
|----------|-----|
| Kubernetes Slack | https://slack.k8s.io |
| r/kubernetes | https://reddit.com/r/kubernetes |
| KubeCon (conference) | https://events.linuxfoundation.org |
| CNCF YouTube | https://youtube.com/@cncf |
| The New Stack (k8s news) | https://thenewstack.io |

## Tools Reference

| Tool | Purpose | Install |
|------|---------|---------|
| `kubectl` | Primary CLI | Included with cluster |
| `k9s` | Terminal UI | `brew install k9s` |
| `kubectx` / `kubens` | Context/namespace switching | `brew install kubectx` |
| `stern` | Multi-pod log tailing | `brew install stern` |
| `kustomize` | Manifest overlays | `brew install kustomize` |
| `helm` | Package manager | `brew install helm` |
| `argocd` CLI | GitOps | `brew install argocd` |
| `flux` CLI | GitOps | `brew install fluxcd/tap/flux` |
| `istioctl` | Istio CLI | `brew install istioctl` |
| `clusterctl` | Cluster API | `brew install clusterctl` |
| `kubeseal` | Sealed Secrets | `brew install kubeseal` |
| `trivy` | Image scanner | `brew install aquasecurity/trivy/trivy` |
| `cosign` | Image signing | `brew install cosign` |
| `kube-bench` | CIS benchmark | GitHub release |
| `kubeadm` | Cluster bootstrap | Package manager |
| `etcdctl` | etcd client | Package manager |
| `crictl` | Container runtime CLI | Package manager |
