---
tags: [kubernetes, prerequisites, linux, phase/0]
phase: 0
topic: Linux
status: not-started
created: 2026-03-01
---

# Linux Basics

Kubernetes runs on Linux. Nodes are Linux machines. Containers share the Linux kernel. You don't need to be a sysadmin, but you need to be comfortable with the fundamentals.

## Key Concepts to Learn

### File System
- Directory hierarchy: `/etc`, `/var`, `/proc`, `/sys`, `/tmp`
- Permissions: `chmod`, `chown`, `umask`
- Symbolic links vs hard links
- Mounting and unmounting (`mount`, `df`, `lsblk`)

### Processes
- `ps aux`, `top`, `htop`
- `kill`, `killall`, signals (`SIGTERM`, `SIGKILL`)
- Background jobs: `&`, `nohup`, `jobs`, `fg`, `bg`
- `systemctl` — start, stop, enable, status services

### Networking Commands
- `ip addr`, `ip route` — interfaces and routing
- `ss` / `netstat` — open ports and connections
- `ping`, `traceroute`, `dig`, `nslookup`
- `curl`, `wget` — HTTP from the command line
- `iptables` basics — how Linux firewall rules work (k8s uses this heavily)

### Shell & Scripting
- Bash: variables, loops, conditionals, functions
- Pipes and redirection: `|`, `>`, `>>`, `2>&1`
- `grep`, `awk`, `sed`, `cut`, `sort`, `uniq`
- `env`, `export`, `source`
- `cron` and `crontab`

### Text Editors
- `vim` basics: insert, save, quit, search, replace
- `nano` as a fallback

## Resources

| Resource | Type | Link |
|----------|------|------|
| The Linux Command Line (Shotts) | Book | https://linuxcommand.org/tlcl.php (free) |
| Linux Journey | Interactive | https://linuxjourney.com |
| OverTheWire: Bandit | CTF/practice | https://overthewire.org/wargames/bandit |
| Ryan's Linux Tutorial | Website | https://ryanstutorials.net/linuxtutorial |

## Practice

- [ ] Navigate the filesystem using only CLI
- [ ] Create a user, set permissions on a directory
- [ ] Write a bash script that tails a log and alerts on a pattern
- [ ] Use `systemctl` to manage a service
- [ ] Inspect open ports with `ss -tlnp`

---

← [[index|Phase 0 Index]] | Next: [[Docker-Fundamentals]] →
