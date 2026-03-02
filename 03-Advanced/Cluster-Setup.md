---
tags: [kubernetes, cluster-setup, kubeadm, k3s, managed, phase/3]
phase: 3
topic: Cluster Setup
status: not-started
created: 2026-03-01
---

# Cluster Setup

Understanding how clusters are built gives you the insight to debug them, secure them, and operate them confidently.

## Options at a Glance

| Option | Best For | Complexity |
|--------|----------|------------|
| `minikube` / `kind` / `k3d` | Local development only | Low |
| `k3s` | Edge, IoT, homelab, small prod | Low |
| `kubeadm` | Learning internals, on-prem | Medium |
| Managed (EKS/GKE/AKS) | Production | Low (control plane managed) |
| Cluster API | Declarative fleet management | High |

## kubeadm — Bootstrap From Scratch

The canonical way to understand what Kubernetes actually needs.

### Prerequisites (on each node)
```bash
# Disable swap
swapoff -a
sed -i '/ swap / s/^/#/' /etc/fstab

# Enable kernel modules
modprobe overlay
modprobe br_netfilter

# Sysctl
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --system

# Install containerd
apt-get install -y containerd
```

### Install kubeadm, kubelet, kubectl
```bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

### Initialize Control Plane
```bash
kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --control-plane-endpoint=k8s-api.example.com

# Save the join command output!

# Configure kubectl
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config

# Install CNI (Flannel)
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

### Join Worker Nodes
```bash
# On each worker, run the join command from kubeadm init output:
kubeadm join k8s-api.example.com:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

### High Availability Control Plane
- Minimum: 3 control plane nodes (for etcd quorum)
- Load balancer in front of API servers
- External etcd cluster (for large deployments)

```bash
# Join additional control plane nodes
kubeadm join k8s-api.example.com:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --control-plane \
  --certificate-key <cert-key>
```

## k3s — Lightweight Kubernetes

Ideal for edge, IoT, homelab, and single-board computers.

```bash
# Single-node install
curl -sfL https://get.k3s.io | sh -

# Multi-node — get server token
cat /var/lib/rancher/k3s/server/node-token

# Join agent node
curl -sfL https://get.k3s.io | \
  K3S_URL=https://server-ip:6443 \
  K3S_TOKEN=<token> sh -

# Use kubectl
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

## Managed Clusters (EKS/GKE/AKS)

The control plane is managed by the cloud provider — you only manage nodes and workloads.

```bash
# EKS
eksctl create cluster --name prod --region us-east-1 --nodes 3

# GKE
gcloud container clusters create prod --num-nodes 3 --zone us-central1-a

# AKS
az aks create --resource-group prod-rg --name prod --node-count 3
```

Key concepts for managed clusters:
- **Node groups / Node pools** — groups of nodes with the same type
- **Managed node groups** — cloud handles patching and lifecycle
- **Fargate / Autopilot** — serverless nodes (no EC2/VMs to manage)
- **IRSA / Workload Identity** — pod-level cloud IAM permissions

## Cluster Upgrade

```bash
# Check available versions
kubeadm upgrade plan

# Upgrade control plane
apt-get install -y kubeadm=1.29.x-00
kubeadm upgrade apply v1.29.x
apt-get install -y kubelet=1.29.x-00 kubectl=1.29.x-00
systemctl restart kubelet

# Drain and upgrade worker nodes one at a time
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data
# upgrade kubelet on worker-1
kubectl uncordon worker-1
```

> [!WARNING]
> Always upgrade one minor version at a time (e.g., 1.27 → 1.28 → 1.29, never skip).

## Resources

| Resource | Link |
|----------|------|
| Kubernetes the Hard Way | https://github.com/kelseyhightower/kubernetes-the-hard-way |
| kubeadm docs | https://kubernetes.io/docs/setup/production-environment/tools/kubeadm |
| k3s docs | https://docs.k3s.io |

## Practice

- [ ] Bootstrap a 3-node cluster with kubeadm (use VMs or cloud instances)
- [ ] Install a CNI plugin and verify node-to-pod connectivity
- [ ] Perform a cluster upgrade (one minor version)
- [ ] Set up a k3s cluster on a Raspberry Pi or low-cost VPS
- [ ] Explore a managed cluster (EKS free tier or GKE Autopilot)

---

← [[index|Phase 3 Index]] | Next: [[Observability]] →
