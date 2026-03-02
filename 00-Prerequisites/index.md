---
tags: [kubernetes, prerequisites, phase/0]
phase: 0
status: not-started
created: 2026-03-01
---

# Phase 0 — Prerequisites

Before touching Kubernetes, you need a solid foundation in the tools it builds on.

## Topics

| Topic | Note | Status |
|-------|------|--------|
| Linux Basics | [[Linux-Basics]] | ⬜ |
| Docker & Containers | [[Docker-Fundamentals]] | ⬜ |
| Networking Basics | [[Networking-Basics]] | ⬜ |
| YAML | [[YAML]] | ⬜ |

## Why This Phase Matters

Kubernetes is a container orchestrator. Without understanding containers, you are memorizing commands without understanding what they do. Without networking, Services and Ingress will feel like magic. Without YAML, you cannot write manifests.

## Time Estimate

- 2–3 weeks at 2 hours/day
- 1–2 weeks at 4 hours/day

## Phase Exit Checklist

- [ ] Can write a `Dockerfile` and build an image
- [ ] Can run a container, expose a port, mount a volume
- [ ] Understand what happens when you type `curl http://example.com`
- [ ] Can read and write YAML without syntax errors
- [ ] Comfortable with basic Linux commands (file system, processes, networking)

---

← [[../ROADMAP|Back to Roadmap]] | Next: [[../01-Core-Concepts/index|Phase 1 – Core Concepts]] →
