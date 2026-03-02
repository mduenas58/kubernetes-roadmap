---
tags: [kubernetes, contributing, open-source, sigs, phase/4]
phase: 4
topic: Contributing
status: not-started
created: 2026-03-01
---

# Contributing to Kubernetes

Contributing to Kubernetes or the CNCF ecosystem is one of the clearest signals of mastery. It also accelerates your learning faster than anything else.

## Where to Start

You do NOT need to contribute to `kubernetes/kubernetes` core (it is one of the largest codebases in existence). The ecosystem is vast:

| Project | Difficulty | Language |
|---------|------------|----------|
| `kubernetes/website` — docs | Easiest | Markdown |
| `helm/helm` | Easy–Medium | Go |
| `argoproj/argo-cd` | Medium | Go, TypeScript |
| `prometheus/prometheus` | Medium | Go |
| `kubernetes-sigs/*` | Medium–Hard | Go |
| `kubernetes/kubernetes` core | Hard | Go |

## SIGs — Special Interest Groups

Kubernetes is governed by SIGs. Each SIG owns a part of the project:

| SIG | Area |
|-----|------|
| sig-apps | Workloads (Deployment, StatefulSet…) |
| sig-network | Services, Ingress, DNS, CNI |
| sig-storage | PV, PVC, CSI |
| sig-security | Pod Security, RBAC, secrets |
| sig-node | kubelet, CRI, node lifecycle |
| sig-api-machinery | API server, CRD, webhooks |
| sig-scheduling | Scheduler, descheduler |
| sig-cli | kubectl |
| sig-docs | Documentation |

Find your SIG: https://github.com/kubernetes/community/blob/master/sig-list.md

## Development Environment Setup

```bash
# Fork and clone kubernetes/kubernetes
git clone https://github.com/YOUR_USER/kubernetes.git
cd kubernetes

# Build everything
make all

# Build a specific component
make WHAT=cmd/kubectl

# Run unit tests
make test

# Run e2e tests (needs a cluster)
make test-e2e

# Run specific tests
go test ./pkg/scheduler/... -v -run TestSchedulerBinding
```

### Using kind for Development

```bash
# Build a kind cluster from local Kubernetes source
kind build node-image
kind create cluster --image kindest/node:latest
```

## Finding Good First Issues

```bash
# Good first issues on GitHub
# Filter: label:"good first issue" is:open

# Kubernetes
https://github.com/kubernetes/kubernetes/issues?q=is:open+label:"good+first+issue"

# kubectl
https://github.com/kubernetes/kubectl/issues?q=is:open+label:"help+wanted"
```

Look for issues labeled:
- `good first issue`
- `help wanted`
- `kind/documentation`
- `kind/bug` (well-scoped bugs)

## KEP — Kubernetes Enhancement Proposals

Large features go through a KEP process (similar to Python PEPs or Rust RFCs).

```
Idea → KEP PR → Review → Alpha → Beta → GA
```

Read existing KEPs to understand how decisions are made:
- https://github.com/kubernetes/enhancements/tree/master/keps

## Pull Request Process

```bash
# 1. Sign the CLA (one-time)
# https://cla.kubernetes.io

# 2. Fork, branch, change
git checkout -b fix/my-bug
# make changes

# 3. Run tests
make verify
make test

# 4. Commit with DCO sign-off
git commit -s -m "fix: correct pod restart count calculation"

# 5. Push and open PR
git push origin fix/my-bug
# Open PR on GitHub

# 6. Respond to /lgtm and /approve from reviewers
# Bot handles CI, labels, and merging
```

## Prow — The Kubernetes CI Bot

Pull requests are managed by Prow bot commands:
| Command | Meaning |
|---------|---------|
| `/lgtm` | Reviewer approves (Looks Good To Me) |
| `/approve` | Owner approves for merge |
| `/hold` | Do not merge yet |
| `/assign @user` | Assign reviewer |
| `/test all` | Re-run all tests |
| `/cc @user` | Request review |

## Contributing Beyond Code

- **Documentation** — often the highest-impact contribution
- **Bug reports** — well-written bug reports with reproductions are valuable
- **Issue triage** — help label and close stale issues
- **Stack Overflow / Slack** — answer questions in `#kubernetes-users`
- **Blog posts** — share what you've learned
- **Talks** — KubeCon, local meetups, company tech blogs

## Resources

| Resource | Link |
|----------|------|
| Kubernetes contributor guide | https://github.com/kubernetes/community/tree/master/contributors/guide |
| Developer guide | https://github.com/kubernetes/community/blob/master/contributors/devel/development.md |
| Kubernetes Slack | https://slack.k8s.io |
| KubeCon talks | https://www.youtube.com/@cncf |

## Practice

- [ ] Fix a typo or improve a doc page in `kubernetes/website`
- [ ] Set up a full Kubernetes dev environment with kind
- [ ] Find and reproduce a `good first issue` — comment on it
- [ ] Read one KEP end-to-end to understand the proposal process
- [ ] Give a lightning talk at a local Kubernetes meetup

---

← [[Internals]] | [[../ROADMAP|Back to Roadmap]] →
