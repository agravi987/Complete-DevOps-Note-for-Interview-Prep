# Prometheus Architecture and Monitoring

## 1. Overview
**Prometheus** is an open-source systems monitoring and alerting toolkit, originally built at SoundCloud. It collects **metrics** (numeric measurements over time) from your applications and infrastructure by **scraping** HTTP endpoints. Unlike traditional push-based monitoring, Prometheus pulls data from targets on its own schedule, making it highly reliable and decoupled.

## 2. Why This Matters
- **Proactive vs. Reactive:** Without monitoring, you find out about a server outage when customers call you. With Prometheus, you get an alert at 3:00 AM the moment CPU hits 95% — before customers notice.
- **Kubernetes-Native:** Prometheus is the de facto standard for Kubernetes cluster monitoring. It integrates with almost every K8s component out of the box.
- **Query Power:** PromQL (Prometheus Query Language) lets you ask deep, precise questions about your infrastructure — "What was the average API latency for the `/checkout` endpoint over the last 5 minutes?"

## 3. Core Concepts

- **Metrics:** Numeric data points measured over time. Every metric has a name and optional key-value labels (e.g., `http_requests_total{method="GET", status="200"}`).
- **Metric Types:**
  | Type | Description | Example |
  |---|---|---|
  | **Counter** | Always increases. Counts events. | `http_requests_total` |
  | **Gauge** | Can go up or down. Current value. | `node_memory_MemFree_bytes` |
  | **Histogram** | Distribution of values in buckets. | `http_request_duration_seconds` |
  | **Summary** | Calculated quantiles (p50, p95, p99). | `rpc_duration_seconds` |

- **Scraping:** Prometheus calls an HTTP endpoint (e.g., `http://app:9090/metrics`) every N seconds and stores the returned metrics.
- **Exporter:** A small agent that translates system metrics into the Prometheus format. Examples:
  - `node_exporter` → Linux CPU, RAM, Disk metrics
  - `blackbox_exporter` → HTTP/TCP endpoint health checks
  - `mysqld_exporter` → MySQL database metrics
- **AlertManager:** Receives alerts fired by Prometheus rules and routes them to destinations like Slack, PagerDuty, or email.
- **PromQL:** The query language for slicing and dicing time-series data.

## 4. Architecture / Workflow

```
   Your Apps / Servers         Prometheus Ecosystem
   ─────────────────           ─────────────────────────────
   Node Exporter  (:9100)  ←── Prometheus Server (scrapes)
   App /metrics   (:8080)  ←── │  (stores TSDB on disk)
   Kubernetes API          ←── │
                               │
                               ├─→ AlertManager ──→ Slack / PagerDuty / Email
                               │
                               └─→ Grafana (queries PromQL → dashboards)
```

## 5. Commands & Practical Usage

**PromQL Queries** (run in the Prometheus UI or Grafana):

Total HTTP request rate per second over last 5 minutes:
```promql
rate(http_requests_total[5m])
```

Current free memory on all nodes in megabytes:
```promql
node_memory_MemFree_bytes / 1024 / 1024
```

CPU usage percentage per node:
```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

95th percentile API response latency:
```promql
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

Count of HTTP 5xx errors over last hour:
```promql
increase(http_requests_total{status=~"5.."}[1h])
```

## 6. Configuration / Code Examples

**`prometheus.yml`** — Main scrape configuration:
```yaml
global:
  scrape_interval: 15s       # Scrape every 15 seconds
  evaluation_interval: 15s   # Evaluate alerting rules every 15 seconds

# Alerting: Where to send fired alerts
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager:9093"]

# Load alerting rules from files
rule_files:
  - "alert_rules.yml"

# Scrape configurations: Who to monitor
scrape_configs:
  # Monitor Prometheus itself
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # Monitor Linux nodes via node_exporter
  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "web-server-1:9100"
          - "web-server-2:9100"
          - "db-server:9100"

  # Automatically discover Kubernetes pods with the right annotations
  - job_name: "kubernetes-pods"
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: "true"
```

**`alert_rules.yml`** — Define alerting conditions:
```yaml
groups:
  - name: infrastructure
    rules:
      # Alert if any instance is down for more than 1 minute
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} has been unreachable for more than 1 minute."

      # Alert if disk usage exceeds 85%
      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 15
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Disk usage is above 85% on {{ $labels.mountpoint }}."
```

## 7. Hands-on Step-by-Step (VERY IMPORTANT)

**Step 1: Run Prometheus + Node Exporter via Docker Compose**

Create `docker-compose.yml`:
```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=15d'

  node_exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'

volumes:
  prometheus_data:
```

**Step 2: Create `prometheus.yml`**
```yaml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["node_exporter:9100"]
```

**Step 3: Launch the stack**
```bash
docker-compose up -d
```

**Step 4: Access the Prometheus UI**
Open `http://localhost:9090` in your browser.

**Step 5: Run a PromQL query**
In the Prometheus query bar, type:
```promql
node_memory_MemAvailable_bytes / 1024 / 1024
```
Click Execute. Switch to the **Graph** tab to see the historical trend.

**Step 6: Explore targets**
Go to `Status → Targets`. You will see the `node_exporter` target with a green `UP` status.

## 8. Common Errors & Troubleshooting

- **Error: Target shows `DOWN` in Prometheus UI**
  - **Issue:** Prometheus cannot reach the scrape target's `/metrics` endpoint.
  - **Fix:** Check that the exporter is running (`docker ps`). Verify the hostname/port in `prometheus.yml`. Check firewall rules — is port 9100 open between Prometheus and the node?

- **Error: `ALERTS` query returns no results even though an issue clearly exists**
  - **Issue:** The alert's `for` duration hasn't elapsed yet, OR the alert expression's PromQL is incorrect.
  - **Fix:** Test the `expr` PromQL in the Prometheus graph UI first to ensure it returns data. Reduce `for` duration to `0s` temporarily for testing.

- **Error: Prometheus running out of disk (`tsdb: out of space`)**
  - **Issue:** The TSDB (Time Series Database) is retaining too much data, growing indefinitely.
  - **Fix:** Set a retention period: `--storage.tsdb.retention.time=15d` limits storage to 15 days of data.

## 9. Best Practices

1. **Use labels wisely.** Labels are powerful but dangerous — each unique label combination creates a new time series. A label containing user IDs or request IDs will cause a "cardinality explosion" that crashes Prometheus.
2. **Set retention limits.** Prometheus is not a long-term storage solution. Keep 15-30 days locally and ship older data to a dedicated long-term store like Thanos or Cortex.
3. **Alert on symptoms, not causes.** Alert on `high error rate` (what the user experiences), not `CPU is high` (which may not affect users). Reduce alert fatigue.

## 10. Interview Questions & Answers

**Q1: How does Prometheus collect metrics — push or pull?**
**A1:** Pull (scrape). Prometheus actively calls each target's HTTP `/metrics` endpoint on a schedule. This makes Prometheus independent of the targets — if Prometheus restarts, targets don't need to be reconfigured. The exception is short-lived jobs, which use the Pushgateway to push metrics.

**Q2: What is an Exporter?**
**A2:** An Exporter is a small process that collects metrics from a system or application that doesn't natively expose Prometheus-format metrics, and translates them into the Prometheus exposition format. Example: `node_exporter` translates Linux kernel stats into Prometheus metrics.

**Q3: What is the difference between a Counter and a Gauge?**
**A3:** A Counter only ever increases monotonically (like a car odometer). It tracks events like request counts or error counts — you use `rate()` to get the per-second rate. A Gauge represents a current value that can go up or down, like memory usage or active connections.

**Q4: What does the `rate()` function do in PromQL?**
**A4:** `rate()` calculates the per-second average increase of a Counter over a specified time window. Since Counters reset to 0 on restart, `rate()` handles resets automatically. Example: `rate(http_requests_total[5m])` = average requests per second over the last 5 minutes.

**Q5: What is AlertManager and how does it relate to Prometheus?**
**A5:** Prometheus evaluates alerting rules and fires alerts to AlertManager. AlertManager then handles the routing, grouping, deduplication, and silencing of those alerts, and delivers them to the correct destination (Slack, PagerDuty, email). They are separate processes with separate configurations.

## 11. Quick Revision Summary
- **Prometheus:** Pull-based metrics collection and alerting.
- **Exporter:** Translates system metrics to Prometheus format. `node_exporter` = Linux metrics.
- **PromQL:** The query language. `rate()` for counters, direct query for gauges.
- **AlertManager:** Routes fired Prometheus alerts to Slack/PagerDuty.
- **Cardinality Warning:** Never use high-cardinality values (user IDs) as labels.

## 12. Official Documentation Links
- [Prometheus Official Documentation](https://prometheus.io/docs/introduction/overview/)
- [PromQL Query Language](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Node Exporter GitHub](https://github.com/prometheus/node_exporter)
