---
tags: [kubernetes, prerequisites, networking, phase/0]
phase: 0
topic: Networking
status: not-started
created: 2026-03-01
---

# Networking Basics

Kubernetes networking is often cited as the hardest part of the system. A solid grounding in fundamentals makes Services, Ingress, DNS, Network Policies, and CNI plugins much easier to reason about.

## Key Concepts to Learn

### OSI Model (Practical Focus)
- Layer 3: IP addressing, subnets, CIDR notation (e.g., `10.0.0.0/16`)
- Layer 4: TCP vs UDP, ports, connections, handshakes
- Layer 7: HTTP/HTTPS, headers, methods, status codes

### IP Addressing
- IPv4 address structure and subnetting
- Private ranges: `10.x.x.x`, `172.16-31.x.x`, `192.168.x.x`
- CIDR: `192.168.1.0/24` means 256 addresses
- `localhost` = `127.0.0.1`, `0.0.0.0` = all interfaces

### DNS
- How DNS resolution works (recursive, authoritative)
- `A`, `CNAME`, `MX`, `TXT` records
- `dig` and `nslookup` for debugging
- Why this matters: Kubernetes has an in-cluster DNS server (`CoreDNS`)

### HTTP & Load Balancing
- Request/response model, headers, bodies
- Status codes: 2xx, 3xx, 4xx, 5xx
- Load balancing algorithms: round-robin, least connections
- Reverse proxies: Nginx, HAProxy

### Firewalls & NAT
- What `iptables` does (Kubernetes uses it for Service routing)
- NAT: translating private IPs to public (important for NodePort/LoadBalancer)
- Ingress vs egress traffic

### TLS/SSL
- How TLS handshake works
- Certificates, CAs, `openssl` basics
- Why this matters: Kubernetes API server uses mTLS everywhere

## Resources

| Resource | Type | Link |
|----------|------|------|
| Networking Fundamentals (Practical Networking) | YouTube | https://www.youtube.com/c/PracticalNetworking |
| Computer Networking: A Top-Down Approach | Book | Standard university textbook |
| Cloudflare Learning Center | Website | https://www.cloudflare.com/learning |
| Julia Evans' Networking Zines | Zines | https://wizardzines.com |

## Practice

- [ ] Subnet a `/24` network into 4 equal subnets by hand
- [ ] Use `dig` to trace DNS resolution for a domain
- [ ] Run Nginx locally and inspect request headers with `curl -v`
- [ ] Use `iptables -L` to list firewall rules on a Linux VM
- [ ] Capture HTTP traffic with `tcpdump` on port 80

---

← [[Docker-Fundamentals]] | Next: [[YAML]] →
