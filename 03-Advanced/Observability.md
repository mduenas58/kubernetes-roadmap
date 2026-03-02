---
tags: [kubernetes, observability, prometheus, grafana, loki, opentelemetry, phase/3]
phase: 3
topic: Observability
status: not-started
created: 2026-03-01
---

# Observability — Metrics, Logs & Traces

The three pillars of observability applied to Kubernetes.

```
Metrics  ──► Prometheus + Grafana
Logs     ──► Loki + Grafana  (or EFK: Elasticsearch + Fluentd + Kibana)
Traces   ──► OpenTelemetry + Jaeger / Tempo
```

## Metrics — Prometheus & Grafana

### Install with kube-prometheus-stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  -f prometheus-values.yaml
```

This installs: Prometheus, Grafana, Alertmanager, Node Exporter, kube-state-metrics.

### PromQL — Prometheus Query Language

```promql
# CPU usage per pod
sum(rate(container_cpu_usage_seconds_total{namespace="production"}[5m])) by (pod)

# Memory usage
container_memory_working_set_bytes{namespace="production"} / 1024 / 1024

# HTTP request rate
rate(http_requests_total{status=~"5.."}[5m])

# Pod restart count
increase(kube_pod_container_status_restarts_total[1h]) > 0

# Node disk pressure
node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} < 0.1
```

### ServiceMonitor — Auto-Discover Metrics

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-metrics
  namespace: monitoring
spec:
  namespaceSelector:
    matchNames: [production]
  selector:
    matchLabels:
      app: my-app
  endpoints:
    - port: metrics
      path: /metrics
      interval: 30s
```

### PrometheusRule — Alerting Rules

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: my-app-alerts
  namespace: monitoring
spec:
  groups:
    - name: my-app
      rules:
        - alert: HighErrorRate
          expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "High error rate on {{ $labels.service }}"
            description: "Error rate is {{ $value | humanize }}/s"
```

### Grafana Dashboards

Popular pre-built dashboards (import by ID):
| Dashboard | ID |
|-----------|-----|
| Kubernetes Cluster Overview | 6417 |
| Node Exporter Full | 1860 |
| Kubernetes Pods | 6336 |
| Nginx Ingress | 9614 |

## Logging — Loki

Loki is like Prometheus, but for logs. It indexes only metadata (labels), not the full log text.

```bash
helm install loki grafana/loki-stack \
  --namespace monitoring \
  --set grafana.enabled=false \
  --set promtail.enabled=true
```

### LogQL — Loki Query Language

```logql
# Filter logs from a pod
{namespace="production", pod=~"web-app-.*"} |= "error"

# Count error rate
rate({namespace="production"} |= "ERROR" [5m])

# Parse JSON logs
{app="api"} | json | status >= 500
```

## Logging — EFK Stack (Alternative)

```
Fluentd / Fluent Bit (DaemonSet) ──► Elasticsearch ──► Kibana
```

Fluent Bit is the modern, lightweight replacement for Fluentd:
```bash
helm install fluent-bit fluent/fluent-bit \
  --namespace logging \
  --set config.outputs="[OUTPUT]\n  Name  es\n  Host  elasticsearch:9200"
```

## Distributed Tracing — OpenTelemetry

```yaml
# Auto-instrument your app (Python/Node/Java/Go/etc.)
# Add OpenTelemetry SDK → sends traces to collector

# OpenTelemetry Collector
helm install otel-collector open-telemetry/opentelemetry-collector \
  --set mode=deployment
```

Backends: **Jaeger** (self-hosted), **Grafana Tempo** (integrates with Grafana), **Zipkin**.

## Key Kubernetes Metrics to Monitor

```promql
# Node CPU
1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (node)

# Pod OOMKills
increase(container_oom_events_total[1h])

# PVC disk usage
kubelet_volume_stats_used_bytes / kubelet_volume_stats_capacity_bytes

# API server latency
histogram_quantile(0.99, rate(apiserver_request_duration_seconds_bucket[5m]))

# etcd leader changes (cluster health signal)
changes(etcd_server_is_leader[1h])
```

## Resources

| Resource | Link |
|----------|------|
| Prometheus docs | https://prometheus.io/docs |
| kube-prometheus-stack | https://github.com/prometheus-community/helm-charts |
| Grafana Loki | https://grafana.com/oss/loki |
| OpenTelemetry | https://opentelemetry.io |

## Practice

- [ ] Install kube-prometheus-stack and access Grafana
- [ ] Import dashboard #1860 and explore node metrics
- [ ] Write a PromQL query that alerts when any pod restarts more than 3 times in an hour
- [ ] Deploy an app that exposes `/metrics` and scrape it with a ServiceMonitor
- [ ] Set up Loki + Promtail and search pod logs from Grafana

---

← [[Cluster-Setup]] | Next: [[CICD]] →
