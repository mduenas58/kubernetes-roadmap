---
tags: [kubernetes, prerequisites, yaml, phase/0]
phase: 0
topic: YAML
status: not-started
created: 2026-03-01
---

# YAML

Every Kubernetes manifest is written in YAML. You need to write and read it fluently, and understand common pitfalls that cause silent failures.

## Key Concepts to Learn

### Syntax Fundamentals
```yaml
# Comments start with #

# Key-value pairs
name: my-app
replicas: 3
enabled: true

# Nested objects (indentation = 2 spaces, never tabs)
metadata:
  name: my-pod
  namespace: default

# Lists
ports:
  - 80
  - 443

# List of objects
containers:
  - name: nginx
    image: nginx:1.25
    ports:
      - containerPort: 80
```

### Data Types
```yaml
string: "hello"            # or just: hello
integer: 42
float: 3.14
boolean: true              # also: false, yes, no, on, off
null_value: null           # or: ~

# Multi-line strings
description: |             # literal block — preserves newlines
  Line one
  Line two

summary: >                 # folded block — newlines become spaces
  This is a long
  single sentence.
```

### Anchors & Aliases (DRY)
```yaml
defaults: &defaults
  restartPolicy: Always
  imagePullPolicy: IfNotPresent

production:
  <<: *defaults            # merge anchor
  replicas: 3

staging:
  <<: *defaults
  replicas: 1
```

### Multiple Documents in One File
```yaml
---                        # document separator
apiVersion: v1
kind: Pod
---
apiVersion: v1
kind: Service
```

## Common Kubernetes YAML Pitfalls

> [!WARNING]
> **Indentation errors are the #1 cause of YAML failures.** Always use 2 spaces, never tabs.

```yaml
# WRONG — tab character
containers:
	- name: app     # tab here will fail

# RIGHT — 2 spaces
containers:
  - name: app
```

> [!WARNING]
> **Booleans and strings look alike.** Quote strings that look like booleans.

```yaml
# "yes"/"no" are booleans in YAML 1.1 (used by many k8s tools)
label: "true"    # string
enabled: true    # boolean
env: "yes"       # string — without quotes this would be boolean true
```

## Tools

| Tool | Purpose |
|------|---------|
| `yamllint` | Lint YAML files for syntax errors |
| `yq` | Query and transform YAML (like `jq` for JSON) |
| VS Code YAML extension | Schema validation in editor |
| https://yaml.org/spec | Official spec |

## Practice

- [ ] Write a YAML file with nested objects, lists, and multi-line strings
- [ ] Use an anchor to avoid repeating common fields
- [ ] Lint it with `yamllint`
- [ ] Convert a simple JSON object to YAML by hand
- [ ] Write your first Kubernetes Pod manifest from scratch

---

← [[Networking-Basics]] | Next Phase: [[../01-Core-Concepts/index|Phase 1 – Core Concepts]] →
