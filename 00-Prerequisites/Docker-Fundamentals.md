---
tags: [kubernetes, prerequisites, docker, containers, phase/0]
phase: 0
topic: Docker
status: not-started
created: 2026-03-01
---

# Docker & Container Fundamentals

Kubernetes orchestrates containers. Understanding containers deeply — not just the Docker CLI — is essential before you can reason about Pods, images, and runtime behavior.

## Key Concepts to Learn

### Container Theory
- What is a container? Namespaces + cgroups + union filesystems
- Containers vs VMs — why containers are lightweight
- OCI standard: image spec and runtime spec
- Container runtimes: `containerd`, `CRI-O` (k8s doesn't use Docker directly)

### Docker Images
- Layers and caching
- Writing efficient `Dockerfile`s
- Multi-stage builds — keep images small
- `.dockerignore`
- Image tagging and registries (Docker Hub, GHCR, ECR)
- `docker build`, `docker pull`, `docker push`, `docker tag`

### Running Containers
- `docker run` flags: `-d`, `-p`, `-v`, `--env`, `--name`, `--rm`, `--network`
- `docker exec` — enter a running container
- `docker logs`, `docker inspect`, `docker stats`
- Lifecycle: created → running → stopped → removed

### Networking
- Bridge, host, none, overlay network modes
- Port publishing: `-p host:container`
- Container DNS — containers resolve each other by name on the same network

### Volumes & Storage
- Bind mounts vs named volumes
- `docker volume create/ls/inspect/rm`
- When to use each

### Docker Compose
- `docker-compose.yml` structure
- `up`, `down`, `logs`, `exec`
- Good practice before Kubernetes: model multi-service apps

## Resources

| Resource | Type | Link |
|----------|------|------|
| Docker official docs | Docs | https://docs.docker.com |
| Play with Docker | Browser lab | https://labs.play-with-docker.com |
| Docker & Kubernetes – Full Course (TechWorld) | YouTube | Search "TechWorld with Nana Docker" |
| Dive (image layer inspector) | Tool | https://github.com/wagoodman/dive |

## Practice

- [ ] Write a `Dockerfile` for a Python web app and build it
- [ ] Run the image, expose a port, verify with `curl`
- [ ] Use a multi-stage build to reduce image size
- [ ] Mount a volume and persist data across container restarts
- [ ] Build a multi-service app with Docker Compose (e.g., app + database)
- [ ] Inspect a running container's processes and filesystem with `docker exec`

## Important Note for Kubernetes

> [!NOTE]
> Kubernetes uses **containerd** or **CRI-O** as its container runtime, not the Docker daemon. However, Docker images follow the OCI standard, so images you build with Docker work perfectly in Kubernetes. The knowledge transfers — the runtime is just different under the hood.

---

← [[Linux-Basics]] | Next: [[Networking-Basics]] →
