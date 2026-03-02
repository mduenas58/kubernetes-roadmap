---
tags: [kubernetes, kubectl, cli, phase/1]
phase: 1
topic: kubectl
status: not-started
created: 2026-03-01
---

# kubectl — The Kubernetes CLI

`kubectl` is your primary interface to a Kubernetes cluster. Mastering it is as important as understanding the concepts.

## Setup & Contexts

```bash
# View config file
cat ~/.kube/config

# List contexts (clusters)
kubectl config get-contexts
kubectl config current-context

# Switch context
kubectl config use-context my-cluster

# Switch namespace (install kubens for this)
kubectl config set-context --current --namespace=staging

# Recommended tools
brew install kubectx    # kubectx + kubens for fast switching
brew install k9s        # terminal UI
```

## Core Command Patterns

```bash
# Get resources
kubectl get <resource>
kubectl get <resource> <name>
kubectl get <resource> -n <namespace>
kubectl get <resource> -A               # all namespaces
kubectl get <resource> -o wide          # extra columns
kubectl get <resource> -o yaml          # full YAML output
kubectl get <resource> -o json | jq .   # JSON + jq

# Describe (human-readable detail + events)
kubectl describe <resource> <name>

# Apply/delete manifests
kubectl apply -f manifest.yaml
kubectl apply -f ./                      # all files in directory
kubectl delete -f manifest.yaml
kubectl delete <resource> <name>

# Edit live
kubectl edit deployment/web-app          # opens $EDITOR
```

## Debugging Commands

```bash
# Logs
kubectl logs <pod>
kubectl logs <pod> -c <container>        # multi-container pod
kubectl logs <pod> --previous            # crashed container
kubectl logs -l app=web-app              # all pods matching label
kubectl logs <pod> -f                    # follow (tail -f)
kubectl stern web-app                    # multi-pod tail (install stern)

# Execute
kubectl exec -it <pod> -- bash
kubectl exec -it <pod> -c <container> -- sh

# Port forward (local debugging)
kubectl port-forward pod/my-pod 8080:80
kubectl port-forward svc/my-svc 8080:80
kubectl port-forward deployment/my-app 8080:80

# Copy files
kubectl cp <pod>:/path/to/file ./local-copy
kubectl cp ./local-file <pod>:/path/in/pod

# Top (metrics-server required)
kubectl top nodes
kubectl top pods
kubectl top pods --sort-by=memory
```

## Imperative Commands (Quick Operations)

```bash
# Create resources without manifests
kubectl create deployment web --image=nginx:1.25 --replicas=3
kubectl create service clusterip web --tcp=80:80
kubectl create configmap app-config --from-literal=ENV=prod
kubectl create secret generic creds --from-literal=password=secret
kubectl create namespace staging

# Generate YAML without applying (—dry-run trick)
kubectl create deployment web --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl expose deployment web --port=80 --dry-run=client -o yaml >> deploy.yaml
```

## Useful Flags

| Flag | Meaning |
|------|---------|
| `-n <ns>` | Target namespace |
| `-A` | All namespaces |
| `-l key=value` | Label selector |
| `--field-selector` | Field selector |
| `-o yaml/json/wide/name` | Output format |
| `--dry-run=client` | Validate without applying |
| `-w` | Watch for changes |
| `--force` | Force delete (use carefully) |
| `--grace-period=0` | Immediate termination |

## JSONPath & Custom Columns

```bash
# Extract specific fields
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase
```

## kubectl Aliases (Add to ~/.bashrc or ~/.zshrc)

```bash
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployments'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'
alias klog='kubectl logs'
alias kex='kubectl exec -it'
alias kpf='kubectl port-forward'
complete -F __start_kubectl k    # tab completion for alias
```

## Resources

| Resource | Link |
|----------|------|
| kubectl cheat sheet | https://kubernetes.io/docs/reference/kubectl/cheatsheet |
| kubectl reference | https://kubernetes.io/docs/reference/kubectl |
| k9s | https://k9scli.io |
| stern | https://github.com/stern/stern |

## Practice

- [ ] Set up tab completion for `kubectl`
- [ ] Use `--dry-run=client -o yaml` to generate manifests for every resource type
- [ ] Debug a crashlooping pod using `logs --previous` and `describe`
- [ ] Use `port-forward` to access a ClusterIP service from your laptop
- [ ] Build a set of aliases you'll actually use

---

← [[Configuration]] | Next Phase: [[../02-Intermediate/index|Phase 2 – Intermediate]] →
