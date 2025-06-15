# Prometheus Cheatsheet

Prometheus is an open-source monitoring and alerting toolkit designed for reliability and
scalability. It collects and stores metrics as time series data, identified by metric name
and key-value pairs called labels.

## Overview

### Key Features

- **Multi-dimensional Data Model** - Time series identified by metric name and labels
- **PromQL** - Flexible query language for data selection and aggregation
- **Pull-based Collection** - HTTP pulls metrics from instrumented targets
- **Service Discovery** - Dynamic target discovery via multiple mechanisms
- **Alerting** - Alertmanager handles alerts sent by Prometheus
- **Visualization** - Built-in expression browser and Grafana integration
- **Storage** - Local on-disk time series database
- **Federation** - Hierarchical federation for scalability

### Core Components

- **Prometheus Server** - Scrapes and stores time series data
- **Client Libraries** - Instrument application code
- **Push Gateway** - For short-lived jobs
- **Exporters** - Expose metrics from third-party systems
- **Alertmanager** - Handles alerts and notifications
- **Grafana** - Visualization and dashboarding

### Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Target    │    │   Target    │    │ Push Gateway│
│ Application │    │  Exporter   │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                    ┌─────────────┐
                    │ Prometheus  │
                    │   Server    │
                    └─────────────┘
                            │
                    ┌─────────────┐    ┌─────────────┐
                    │ Alertmanager│    │   Grafana   │
                    └─────────────┘    └─────────────┘
```

## Installation

### Docker Installation

```bash
# Run Prometheus server
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

# Run with data persistence
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v prometheus-data:/prometheus \
  prom/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/prometheus \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.console.templates=/etc/prometheus/consoles \
  --web.enable-lifecycle
```

### Binary Installation

```bash
# Download and install Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvfz prometheus-2.45.0.linux-amd64.tar.gz
cd prometheus-2.45.0.linux-amd64

# Run Prometheus
./prometheus --config.file=prometheus.yml
```

### Kubernetes Installation with Helm

```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/prometheus

# Install kube-prometheus-stack (Prometheus + Grafana + Alertmanager)
helm install monitoring prometheus-community/kube-prometheus-stack

# Custom values
helm install prometheus prometheus-community/prometheus \
  --set server.persistentVolume.size=20Gi \
  --set alertmanager.enabled=false
```

### macOS Installation

```bash
# Install with Homebrew
brew install prometheus

# Start Prometheus
brew services start prometheus

# Or run directly
prometheus --config.file=/usr/local/etc/prometheus.yml
```

## Configuration

### Basic prometheus.yml

```yaml
# prometheus.yml
global:
  scrape_interval: 15s      # Set the scrape interval to every 15 seconds
  evaluation_interval: 15s  # Evaluate rules every 15 seconds
  external_labels:
    cluster: 'production'
    region: 'us-west-1'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

# Load rules once and periodically evaluate them
rule_files:
  - "rules/*.yml"
  - "alerts/*.yml"

# Scrape configuration
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node exporter
  - job_name: 'node-exporter'
    static_configs:
      - targets: 
        - 'node1:9100'
        - 'node2:9100'
        - 'node3:9100'
    scrape_interval: 10s
    metrics_path: /metrics
    scheme: http

  # Application metrics
  - job_name: 'myapp'
    static_configs:
      - targets: ['app1:8080', 'app2:8080']
    metrics_path: /actuator/prometheus
    scrape_interval: 30s
```

### Advanced Configuration

```yaml
# Advanced prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  scrape_timeout: 10s
  external_labels:
    monitor: 'production'

# Remote write configuration
remote_write:
  - url: "https://prometheus-remote-write.example.com/api/v1/write"
    basic_auth:
      username: user
      password: pass
    write_relabel_configs:
      - source_labels: [__name__]
        regex: 'go_.*'
        action: drop

# Remote read configuration
remote_read:
  - url: "https://prometheus-remote-read.example.com/api/v1/read"
    read_recent: true

scrape_configs:
  # Kubernetes service discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name

  # File-based service discovery
  - job_name: 'file-sd'
    file_sd_configs:
      - files:
        - '/etc/prometheus/targets/*.json'
        - '/etc/prometheus/targets/*.yml'
    refresh_interval: 10s

  # HTTP service discovery
  - job_name: 'http-sd'
    http_sd_configs:
      - url: 'http://service-discovery:8080/targets'
        refresh_interval: 30s

  # Consul service discovery
  - job_name: 'consul'
    consul_sd_configs:
      - server: 'consul:8500'
        services: ['web', 'api']
    relabel_configs:
      - source_labels: ['__meta_consul_tags']
        regex: '.*,prometheus,.*'
        action: keep
```

### Service Discovery Files

```json
# targets.json
[
  {
    "targets": ["host1:9100", "host2:9100"],
    "labels": {
      "job": "node-exporter",
      "env": "production"
    }
  },
  {
    "targets": ["app1:8080", "app2:8080"],
    "labels": {
      "job": "webapp",
      "env": "production",
      "version": "v1.2.3"
    }
  }
]
```

```yaml
# targets.yml
- targets:
  - 'db1:9104'
  - 'db2:9104'
  labels:
    job: 'mysql-exporter'
    env: 'production'
    cluster: 'main'
```

## PromQL (Prometheus Query Language)

### Basic Syntax

```promql
# Instant vector - single value per time series at one point in time
http_requests_total

# Range vector - set of time series containing range of data points over time
http_requests_total[5m]

# Scalar - simple numeric floating point value
42

# String literal
"hello world"
```

### Selectors and Matchers

```promql
# Label matching
http_requests_total{method="GET"}
http_requests_total{method!="GET"}
http_requests_total{method=~"GET|POST"}
http_requests_total{method!~"GET|POST"}

# Multiple labels
http_requests_total{method="GET", status="200"}

# All time series with specific metric name
up

# Empty label matcher
{__name__="http_requests_total"}
```

### Time Range Selectors

```promql
# Range vectors
http_requests_total[5m]    # Last 5 minutes
http_requests_total[1h]    # Last 1 hour
http_requests_total[1d]    # Last 1 day
http_requests_total[1w]    # Last 1 week

# Time units: s (seconds), m (minutes), h (hours), d (days), w (weeks), y (years)
```

### Operators

```promql
# Arithmetic operators
node_memory_MemTotal_bytes / 1024 / 1024  # Convert to MB
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# Comparison operators
up == 1              # Equal
up != 1              # Not equal
cpu_usage > 80       # Greater than
cpu_usage < 20       # Less than
cpu_usage >= 80      # Greater than or equal
cpu_usage <= 20      # Less than or equal

# Logical operators
up == 1 and cpu_usage < 80      # AND
up == 0 or cpu_usage > 90       # OR
not (up == 1)                   # NOT
```

### Aggregation Operators

```promql
# Sum
sum(http_requests_total)
sum(http_requests_total) by (method)
sum(http_requests_total) without (instance)

# Count
count(up)
count(up == 1)
count by (job) (up)

# Average
avg(cpu_usage)
avg by (instance) (cpu_usage)

# Min/Max
min(cpu_usage)
max(cpu_usage)
min by (job) (up)

# Grouping
group(up)
group by (job) (up)

# Standard deviation
stddev(cpu_usage)
stdvar(cpu_usage)

# Quantiles
quantile(0.95, http_request_duration_seconds)
quantile by (job) (0.99, http_request_duration_seconds)

# Top/Bottom K
topk(5, http_requests_total)
bottomk(3, cpu_usage)
```

### Functions

```promql
# Rate and increase
rate(http_requests_total[5m])           # Per-second rate
irate(http_requests_total[5m])          # Instant rate
increase(http_requests_total[1h])       # Increase over time

# Delta and deriv
delta(cpu_temp_celsius[2h])             # Difference between first and last value
deriv(node_memory_MemFree_bytes[1h])    # Per-second derivative

# Predict and trend
predict_linear(node_filesystem_free_bytes[1h], 4*3600)  # Predict 4 hours ahead

# Time functions
time()                                  # Current Unix timestamp
hour()                                  # Hour of day (0-23)
day_of_month()                          # Day of month (1-31)
day_of_week()                           # Day of week (0-6, Sunday=0)
month()                                 # Month (1-12)
year()                                  # Year

# Math functions
abs(cpu_temp_celsius - 70)              # Absolute value
round(cpu_usage, 0.1)                   # Round to nearest 0.1
floor(cpu_usage)                        # Round down
ceil(cpu_usage)                         # Round up
sqrt(value)                             # Square root
exp(value)                              # Exponential
ln(value)                               # Natural logarithm
log2(value)                             # Base-2 logarithm
log10(value)                            # Base-10 logarithm

# String functions
label_replace(up, "new_label", "$1", "instance", "([^:]+):.*")
label_join(up, "endpoint", ":", "instance", "job")

# Histogram functions
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### Advanced Queries

```promql
# Memory usage percentage
(
  node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
) / node_memory_MemTotal_bytes * 100

# CPU usage percentage (from idle)
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Disk usage percentage
(
  node_filesystem_size_bytes{fstype!="tmpfs"} - 
  node_filesystem_free_bytes{fstype!="tmpfs"}
) / node_filesystem_size_bytes{fstype!="tmpfs"} * 100

# Request rate per minute
sum(rate(http_requests_total[5m])) * 60

# Error rate percentage
sum(rate(http_requests_total{status=~"5.."}[5m])) / 
sum(rate(http_requests_total[5m])) * 100

# Average response time
sum(rate(http_request_duration_seconds_sum[5m])) / 
sum(rate(http_request_duration_seconds_count[5m]))

# P95 response time
histogram_quantile(0.95, 
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)

# Top 5 URLs by request count
topk(5, sum by (url) (rate(http_requests_total[5m])))

# Instances with high CPU for more than 5 minutes
avg_over_time(cpu_usage[5m]) > 80

# Join metrics from different jobs
node_load1 * on(instance) group_left(job) up{job="node-exporter"}
```

## Metrics Types

### Counter

```promql
# Always increasing value (e.g., requests, errors)
http_requests_total
node_network_receive_bytes_total

# Use rate() or increase() with counters
rate(http_requests_total[5m])      # Requests per second
increase(http_requests_total[1h])  # Total requests in last hour
```

### Gauge

```promql
# Current value that can go up or down (e.g., temperature, memory)
node_memory_MemAvailable_bytes
cpu_temperature_celsius
up

# Use directly or with aggregation
avg(cpu_temperature_celsius)
max(node_memory_MemAvailable_bytes)
```

### Histogram

```promql
# Distribution of observations (e.g., request duration, response size)
http_request_duration_seconds_bucket
http_request_duration_seconds_sum
http_request_duration_seconds_count

# Calculate percentiles
histogram_quantile(0.95, 
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)

# Average duration
sum(rate(http_request_duration_seconds_sum[5m])) / 
sum(rate(http_request_duration_seconds_count[5m]))
```

### Summary

```promql
# Similar to histogram but calculates quantiles on client side
http_request_duration_seconds{quantile="0.95"}
http_request_duration_seconds_sum
http_request_duration_seconds_count

# Use quantile values directly
http_request_duration_seconds{quantile="0.99"}
```

## Recording Rules

### Basic Recording Rules

```yaml
# rules/recording.yml
groups:
  - name: cpu_rules
    interval: 30s
    rules:
      - record: instance:cpu_usage:rate5m
        expr: |
          100 - (avg by (instance) (
            rate(node_cpu_seconds_total{mode="idle"}[5m])
          ) * 100)
        labels:
          team: infrastructure

      - record: job:cpu_usage:mean5m
        expr: avg by (job) (instance:cpu_usage:rate5m)

  - name: memory_rules
    rules:
      - record: instance:memory_usage:ratio
        expr: |
          (
            node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
          ) / node_memory_MemTotal_bytes

      - record: instance:memory_usage:percentage
        expr: instance:memory_usage:ratio * 100

  - name: http_rules
    rules:
      - record: job:http_requests:rate5m
        expr: |
          sum(rate(http_requests_total[5m])) by (job)

      - record: job:http_errors:rate5m
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) by (job)

      - record: job:http_error_rate:ratio5m
        expr: |
          job:http_errors:rate5m / job:http_requests:rate5m
```

### Advanced Recording Rules

```yaml
# rules/sli.yml
groups:
  - name: sli_rules
    interval: 10s
    rules:
      # Availability SLI
      - record: sli:availability:ratio1m
        expr: |
          (
            sum(rate(http_requests_total{status!~"5.."}[1m])) by (service)
          ) / (
            sum(rate(http_requests_total[1m])) by (service)
          )

      # Latency SLI (P99)
      - record: sli:latency:p99_1m
        expr: |
          histogram_quantile(0.99,
            sum(rate(http_request_duration_seconds_bucket[1m])) by (service, le)
          )

      # Throughput SLI
      - record: sli:throughput:rate1m
        expr: |
          sum(rate(http_requests_total[1m])) by (service)

      # Error budget burn rate
      - record: sli:error_budget_burn_rate:1h
        expr: |
          (
            1 - sli:availability:ratio1m
          ) / (1 - 0.99)  # Assuming 99% SLO
```

## Alerting Rules

### Basic Alert Rules

```yaml
# rules/alerts.yml
groups:
  - name: instance_alerts
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
          team: infrastructure
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
          runbook_url: "https://runbooks.example.com/instance-down"

      - alert: HighCPUUsage
        expr: instance:cpu_usage:rate5m > 80
        for: 10m
        labels:
          severity: warning
          team: infrastructure
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"

      - alert: HighMemoryUsage
        expr: instance:memory_usage:percentage > 90
        for: 5m
        labels:
          severity: critical
          team: infrastructure
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}% on {{ $labels.instance }}"

      - alert: DiskSpaceLow
        expr: |
          (
            node_filesystem_size_bytes{fstype!="tmpfs"} - 
            node_filesystem_free_bytes{fstype!="tmpfs"}
          ) / node_filesystem_size_bytes{fstype!="tmpfs"} * 100 > 85
        for: 5m
        labels:
          severity: warning
          team: infrastructure
        annotations:
          summary: "Disk space low on {{ $labels.instance }}"
          description: "Disk usage is {{ $value }}% on {{ $labels.instance }} {{ $labels.mountpoint }}"
```

### Application Alert Rules

```yaml
# rules/app_alerts.yml
groups:
  - name: http_alerts
    rules:
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
          ) / (
            sum(rate(http_requests_total[5m])) by (service)
          ) * 100 > 5
        for: 2m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate for {{ $labels.service }}"
          description: "Error rate is {{ $value }}% for service {{ $labels.service }}"

      - alert: HighLatency
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (service, le)
          ) > 0.5
        for: 5m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "High latency for {{ $labels.service }}"
          description: "95th percentile latency is {{ $value }}s for service {{ $labels.service }}"

      - alert: LowThroughput
        expr: |
          sum(rate(http_requests_total[5m])) by (service) < 10
        for: 10m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "Low throughput for {{ $labels.service }}"
          description: "Throughput is {{ $value }} req/s for service {{ $labels.service }}"

  - name: database_alerts
    rules:
      - alert: DatabaseConnectionsHigh
        expr: mysql_global_status_threads_connected > 80
        for: 5m
        labels:
          severity: warning
          team: database
        annotations:
          summary: "High database connections"
          description: "Database has {{ $value }} active connections"

      - alert: SlowQueries
        expr: rate(mysql_global_status_slow_queries[5m]) > 1
        for: 5m
        labels:
          severity: warning
          team: database
        annotations:
          summary: "Slow queries detected"
          description: "{{ $value }} slow queries per second detected"
```

### SLO-based Alerts

```yaml
# rules/slo_alerts.yml
groups:
  - name: slo_alerts
    rules:
      # Multi-window, multi-burn-rate alerts
      - alert: AvailabilitySLOBurnRateCritical
        expr: |
          (
            (
              1 - sli:availability:ratio1m
            ) / (1 - 0.99)
          ) > 14.4  # 14.4x burn rate for 1h
          and
          (
            (
              1 - avg_over_time(sli:availability:ratio1m[5m])
            ) / (1 - 0.99)
          ) > 14.4
        labels:
          severity: critical
          slo: availability
        annotations:
          summary: "Critical availability SLO burn rate"
          description: "Availability SLO is burning at {{ $value }}x the acceptable rate"

      - alert: AvailabilitySLOBurnRateHigh
        expr: |
          (
            (
              1 - avg_over_time(sli:availability:ratio1m[5m])
            ) / (1 - 0.99)
          ) > 6  # 6x burn rate for 6h
          and
          (
            (
              1 - avg_over_time(sli:availability:ratio1m[30m])
            ) / (1 - 0.99)
          ) > 6
        labels:
          severity: warning
          slo: availability
        annotations:
          summary: "High availability SLO burn rate"
          description: "Availability SLO is burning at {{ $value }}x the acceptable rate"

      - alert: LatencySLOBreach
        expr: sli:latency:p99_1m > 0.5  # 500ms SLO
        for: 2m
        labels:
          severity: warning
          slo: latency
        annotations:
          summary: "Latency SLO breach"
          description: "P99 latency is {{ $value }}s, exceeding SLO of 500ms"
```

## Exporters

### Node Exporter

```bash
# Install Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.6.0.linux-amd64.tar.gz
cd node_exporter-1.6.0.linux-amd64

# Run Node Exporter
./node_exporter

# With custom collectors
./node_exporter --collector.systemd --collector.processes

# Disable collectors
./node_exporter --no-collector.wifi --no-collector.hwmon
```

```yaml
# Docker Compose
version: '3'
services:
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped
```

### Application Metrics

```python
# Python application with prometheus_client
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time
import random

# Metrics
REQUESTS_TOTAL = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint', 'status'])
REQUEST_DURATION = Histogram('http_request_duration_seconds', 'HTTP request duration', ['method', 'endpoint'])
ACTIVE_CONNECTIONS = Gauge('active_connections', 'Active connections')

def handle_request(method, endpoint):
    ACTIVE_CONNECTIONS.inc()
    with REQUEST_DURATION.labels(method=method, endpoint=endpoint).time():
        # Simulate request processing
        time.sleep(random.uniform(0.1, 0.5))
        
        # Simulate status codes
        status = '200' if random.random() > 0.1 else '500'
        REQUESTS_TOTAL.labels(method=method, endpoint=endpoint, status=status).inc()
    
    ACTIVE_CONNECTIONS.dec()
    return status

if __name__ == '__main__':
    # Start Prometheus metrics server
    start_http_server(8000)
    
    # Simulate application
    while True:
        handle_request('GET', '/api/users')
        time.sleep(random.uniform(0.1, 1.0))
```

```go
// Go application with prometheus/client_golang
package main

import (
    "log"
    "math/rand"
    "net/http"
    "time"
    
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )
    
    httpRequestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "Duration of HTTP requests",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "endpoint"},
    )
    
    activeConnections = prometheus.NewGauge(
        prometheus.GaugeOpts{
            Name: "active_connections",
            Help: "Number of active connections",
        },
    )
)

func init() {
    prometheus.MustRegister(httpRequestsTotal)
    prometheus.MustRegister(httpRequestDuration)
    prometheus.MustRegister(activeConnections)
}

func handler(w http.ResponseWriter, r *http.Request) {
    activeConnections.Inc()
    defer activeConnections.Dec()
    
    timer := prometheus.NewTimer(httpRequestDuration.WithLabelValues(r.Method, r.URL.Path))
    defer timer.ObserveDuration()
    
    // Simulate processing
    time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
    
    status := "200"
    if rand.Float32() < 0.1 {
        status = "500"
        w.WriteHeader(500)
    }
    
    httpRequestsTotal.WithLabelValues(r.Method, r.URL.Path, status).Inc()
}

func main() {
    http.HandleFunc("/api/users", handler)
    http.Handle("/metrics", promhttp.Handler())
    
    log.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### Common Exporters

```bash
# MySQL Exporter
docker run -d \
  --name mysql-exporter \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="user:password@(mysql:3306)/" \
  prom/mysqld-exporter

# PostgreSQL Exporter
docker run -d \
  --name postgres-exporter \
  -p 9187:9187 \
  -e DATA_SOURCE_NAME="postgresql://username:password@postgres:5432/database?sslmode=disable" \
  wrouesnel/postgres_exporter

# Redis Exporter
docker run -d \
  --name redis-exporter \
  -p 9121:9121 \
  oliver006/redis_exporter \
  --redis.addr redis://redis:6379

# Nginx Exporter
docker run -d \
  --name nginx-exporter \
  -p 9113:9113 \
  nginx/nginx-prometheus-exporter:latest \
  -nginx.scrape-uri=http://nginx:8080/stub_status

# Blackbox Exporter
docker run -d \
  --name blackbox-exporter \
  -p 9115:9115 \
  -v /path/to/blackbox.yml:/etc/blackbox_exporter/config.yml \
  prom/blackbox-exporter
```

## Kubernetes Integration

### Kubernetes Service Discovery

```yaml
# kubernetes.yml
scrape_configs:
  # Kubernetes API server
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

  # Kubernetes nodes
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics

  # Kubernetes pods
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name

  # Kubernetes services
  - job_name: 'kubernetes-service-endpoints'
    kubernetes_sd_configs:
      - role: endpoints
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name
```

### ServiceMonitor (Prometheus Operator)

```yaml
# servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-metrics
  namespace: monitoring
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
    scheme: http
  namespaceSelector:
    matchNames:
    - default
    - production
```

### PodMonitor (Prometheus Operator)

```yaml
# podmonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: my-app-pods
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: my-app
  podMetricsEndpoints:
  - port: metrics
    interval: 15s
    path: /metrics
    scheme: http
```

### PrometheusRule (Prometheus Operator)

```yaml
# prometheusrule.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: my-app-rules
  namespace: monitoring
spec:
  groups:
  - name: my-app.rules
    rules:
    - alert: MyAppDown
      expr: up{job="my-app"} == 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "My App is down"
        description: "My App has been down for more than 5 minutes."
    
    - record: my_app:request_rate5m
      expr: sum(rate(http_requests_total{job="my-app"}[5m]))
```

## Federation

### Basic Federation

```yaml
# Global Prometheus configuration
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{job=~"prometheus|node-exporter"}'  # Federate specific jobs
        - '{__name__=~"job:.*"}' # Federate recording rule results
        - 'up'
    static_configs:
      - targets:
        - 'prometheus-dc1:9090'
        - 'prometheus-dc2:9090'
        - 'prometheus-k8s:9090'
```

### Hierarchical Federation

```yaml
# Regional Prometheus federating from local instances
scrape_configs:
  - job_name: 'federate-local'
    scrape_interval: 15s
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - 'up{job=~"prometheus|alertmanager"}'  # Infrastructure metrics
        - '{__name__=~"instance:.*"}' # Instance-level aggregations
        - '{__name__=~"job:.*"}'      # Job-level aggregations
    static_configs:
      - targets:
        - 'prometheus-zone-a:9090'
        - 'prometheus-zone-b:9090'
        - 'prometheus-zone-c:9090'
    relabel_configs:
      - source_labels: [__address__]
        regex: 'prometheus-zone-(.):.+'
        target_label: zone
        replacement: '${1}'

# Global Prometheus federating from regional instances
  - job_name: 'federate-regional'
    scrape_interval: 30s
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - 'up{job=~"prometheus|alertmanager"}'  # Infrastructure metrics
        - '{__name__=~"job:.*"}'       # Job-level aggregations
        - '{__name__=~"cluster:.*"}'   # Cluster-level aggregations
    static_configs:
      - targets:
        - 'prometheus-us-west:9090'
        - 'prometheus-us-east:9090'
        - 'prometheus-eu-west:9090'
```

## HTTP API

### Query API

```bash
# Instant query
curl 'http://localhost:9090/api/v1/query?query=up'

# Range query
curl 'http://localhost:9090/api/v1/query_range?query=up&start=2023-01-01T00:00:00Z&end=2023-01-01T01:00:00Z&step=15s'

# Label values
curl 'http://localhost:9090/api/v1/label/job/values'

# Series
curl 'http://localhost:9090/api/v1/series?match[]=up'

# Targets
curl 'http://localhost:9090/api/v1/targets'

# Rules
curl 'http://localhost:9090/api/v1/rules'

# Alerts
curl 'http://localhost:9090/api/v1/alerts'

# Configuration
curl 'http://localhost:9090/api/v1/status/config'

# Flags
curl 'http://localhost:9090/api/v1/status/flags'

# Build info
curl 'http://localhost:9090/api/v1/status/buildinfo'
```

### Administrative API

```bash
# Reload configuration (requires --web.enable-lifecycle)
curl -X POST http://localhost:9090/-/reload

# Quit (requires --web.enable-lifecycle)
curl -X POST http://localhost:9090/-/quit

# Health check
curl http://localhost:9090/-/healthy

# Ready check
curl http://localhost:9090/-/ready

# TSDB stats
curl 'http://localhost:9090/api/v1/status/tsdb'

# WAL replay status
curl 'http://localhost:9090/api/v1/status/walreplay'
```

## Storage and Retention

### Local Storage Configuration

```bash
# Start Prometheus with custom storage settings
prometheus \
  --config.file=prometheus.yml \
  --storage.tsdb.path=./data \
  --storage.tsdb.retention.time=15d \
  --storage.tsdb.retention.size=50GB \
  --storage.tsdb.min-block-duration=2h \
  --storage.tsdb.max-block-duration=25h \
  --storage.tsdb.wal-compression
```

### Remote Storage

```yaml
# Remote write configuration
remote_write:
  - url: "https://prometheus-remote-write.example.com/api/v1/write"
    remote_timeout: 30s
    basic_auth:
      username: user
      password: pass
    write_relabel_configs:
      - source_labels: [__name__]
        regex: 'go_.*'
        action: drop
    queue_config:
      capacity: 10000
      max_shards: 200
      min_shards: 1
      max_samples_per_send: 1000
      batch_send_deadline: 5s
      min_backoff: 30ms
      max_backoff: 100ms

# Remote read configuration
remote_read:
  - url: "https://prometheus-remote-read.example.com/api/v1/read"
    remote_timeout: 1m
    basic_auth:
      username: user
      password: pass
    read_recent: true
```

### TSDB Administration

```bash
# Compact TSDB blocks
promtool tsdb create-blocks-from openmetrics \
  --time.min=2023-01-01T00:00:00Z \
  --time.max=2023-01-02T00:00:00Z \
  input.txt output_directory

# Analyze TSDB
promtool tsdb analyze /path/to/prometheus/data

# Dump TSDB samples
promtool tsdb dump /path/to/prometheus/data \
  --min-time=2023-01-01T00:00:00Z \
  --max-time=2023-01-01T01:00:00Z

# List TSDB blocks
promtool tsdb list /path/to/prometheus/data

# Benchmark TSDB
promtool tsdb bench write \
  --out=data \
  --metrics=1000 \
  --scrapes=100
```

## Monitoring Prometheus

### Key Metrics to Monitor

```promql
# Prometheus process metrics
prometheus_build_info
prometheus_config_last_reload_success_timestamp_seconds
prometheus_config_last_reload_successful

# TSDB metrics
prometheus_tsdb_head_samples_appended_total
prometheus_tsdb_head_series
prometheus_tsdb_compaction_duration_seconds
prometheus_tsdb_wal_fsync_duration_seconds
prometheus_tsdb_retention_limit_bytes

# Scrape metrics
prometheus_target_scrapes_total
prometheus_target_scrape_duration_seconds
prometheus_target_scrapes_exceeded_sample_limit_total

# Query metrics
prometheus_engine_query_duration_seconds
prometheus_engine_queries
prometheus_engine_queries_concurrent_max

# Rule evaluation metrics
prometheus_rule_evaluation_duration_seconds
prometheus_rule_evaluations_total
prometheus_rule_group_duration_seconds

# Remote write metrics
prometheus_remote_storage_samples_total
prometheus_remote_storage_dropped_samples_total
prometheus_remote_storage_failed_samples_total
prometheus_remote_storage_retried_samples_total
prometheus_remote_storage_sent_batch_duration_seconds
prometheus_remote_storage_queue_length
```

### Prometheus Health Checks

```promql
# Config reload failures
time() - prometheus_config_last_reload_success_timestamp_seconds > 300

# High memory usage
process_resident_memory_bytes{job="prometheus"} / 1024 / 1024 / 1024 > 8

# High CPU usage
rate(process_cpu_seconds_total{job="prometheus"}[5m]) * 100 > 80

# TSDB compaction issues
increase(prometheus_tsdb_compactions_failed_total[1h]) > 0

# WAL corruption
increase(prometheus_tsdb_wal_corruptions_total[1h]) > 0

# Too many series
prometheus_tsdb_head_series > 1000000

# Scrape failures
up == 0

# Remote write failures
rate(prometheus_remote_storage_failed_samples_total[5m]) > 0
```

## Performance Tuning

### Query Optimization

```promql
# Use recording rules for expensive queries
# Instead of:
sum(rate(http_requests_total[5m])) by (job)

# Create recording rule:
record: job:http_requests:rate5m
expr: sum(rate(http_requests_total[5m])) by (job)

# Then use:
job:http_requests:rate5m

# Avoid high cardinality in recording rules
# Bad:
sum(rate(http_requests_total[5m])) by (instance, job, method, status)

# Better:
sum(rate(http_requests_total[5m])) by (job)

# Use appropriate range selectors
# For counters, use range that's 2-3x scrape interval
rate(http_requests_total[1m])  # if scrape_interval is 15s

# Use on() and ignoring() to control matching
node_load1 * on(instance) group_left(job) up{job="node-exporter"}
```

### Resource Configuration

```bash
# Memory optimization
prometheus \
  --storage.tsdb.retention.size=50GB \
  --query.max-concurrency=20 \
  --query.max-samples=50000000

# CPU optimization
prometheus \
  --query.timeout=2m \
  --web.max-connections=512

# Storage optimization
prometheus \
  --storage.tsdb.min-block-duration=2h \
  --storage.tsdb.max-block-duration=25h \
  --storage.tsdb.wal-compression
```

### Relabeling Optimization

```yaml
# Drop unwanted metrics early
scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
    metric_relabel_configs:
      # Drop filesystem metrics for tmpfs
      - source_labels: [__name__, fstype]
        regex: 'node_filesystem_.+;tmpfs'
        action: drop
      
      # Keep only specific metrics
      - source_labels: [__name__]
        regex: '(node_cpu_seconds_total|node_memory_MemTotal_bytes|node_memory_MemAvailable_bytes)'
        action: keep
      
      # Rename metric
      - source_labels: [__name__]
        regex: 'node_cpu_seconds_total'
        target_label: __name__
        replacement: 'cpu_seconds_total'
```

## Troubleshooting

### Common Issues

```bash
# Check Prometheus configuration
promtool check config prometheus.yml

# Check rules syntax
promtool check rules rules/*.yml

# Query Prometheus logs
docker logs prometheus

# Check target discovery
curl http://localhost:9090/api/v1/targets

# Check metric ingestion
curl 'http://localhost:9090/api/v1/query?query=prometheus_tsdb_head_samples_appended_total'

# Check for high cardinality
curl 'http://localhost:9090/api/v1/label/__name__/values' | jq '.data | length'

# Find metrics with high cardinality
promtool query instant 'topk(10, count by (__name__)({__name__=~".+"}))'

# Check TSDB stats
curl 'http://localhost:9090/api/v1/status/tsdb'

# Analyze TSDB blocks
promtool tsdb analyze /prometheus/data
```

### Debug Queries

```promql
# Find missing metrics
absent(up{job="my-service"})

# Check label values
label_replace(up, "debug", "$1", "instance", "(.+)")

# Detect time series churn
increase(prometheus_tsdb_head_series_created_total[1h])

# Find slow queries
topk(10, prometheus_engine_query_duration_seconds{quantile="0.9"})

# Check scrape durations
topk(10, scrape_duration_seconds)

# Identify dropped samples
increase(prometheus_target_scrapes_exceeded_sample_limit_total[1h])
```

### Memory Issues

```bash
# Monitor memory usage
echo 'process_resident_memory_bytes{job="prometheus"}' | promtool query instant

# Check head series count
echo 'prometheus_tsdb_head_series' | promtool query instant

# Find high cardinality metrics
promtool tsdb analyze /prometheus/data 2>&1 | grep "metric names with highest sample count"

# Compact TSDB manually
promtool tsdb create-blocks-from openmetrics input.txt /prometheus/data
```

## Best Practices

### Naming Conventions

```promql
# Metric names
http_requests_total          # Counter with _total suffix
http_request_duration_seconds # Use base units (seconds, bytes)
node_memory_MemTotal_bytes   # Include unit in name

# Label names
{method="GET", status="200", endpoint="/api/users"}

# Recording rule names
instance:cpu_usage:rate5m    # instance-level aggregation
job:cpu_usage:rate5m         # job-level aggregation
cluster:cpu_usage:rate5m     # cluster-level aggregation

# Alert names
InstanceDown                 # PascalCase
HighCPUUsage                # Descriptive and specific
DatabaseConnectionsHigh     # Clear and actionable
```

### Configuration Best Practices

```yaml
# Use appropriate scrape intervals
scrape_configs:
  - job_name: 'fast-metrics'
    scrape_interval: 5s      # For critical, fast-changing metrics
  
  - job_name: 'normal-metrics'
    scrape_interval: 15s     # Default for most metrics
  
  - job_name: 'slow-metrics'
    scrape_interval: 60s     # For slow-changing infrastructure metrics

# Use relabeling to control cardinality
metric_relabel_configs:
  - source_labels: [__name__]
    regex: 'high_cardinality_metric.*'
    action: drop
  
  - source_labels: [user_id]
    target_label: user_type
    regex: '^(premium|basic)_.*'
    replacement: '${1}'
```

### Alerting Best Practices

```yaml
# Use appropriate severities
labels:
  severity: critical     # Requires immediate attention
  severity: warning      # Requires attention soon
  severity: info         # Informational

# Include runbook URLs
annotations:
  runbook_url: "https://runbooks.example.com/instance-down"
  
# Use templating in annotations
annotations:
  summary: "High CPU usage on {{ $labels.instance }}"
  description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"
  
# Set appropriate for clauses
for: 5m    # Avoid flapping alerts
for: 0s    # For critical issues that need immediate alerting
```

---

*For more detailed information, visit the [official Prometheus documentation](https://prometheus.io/docs/)*
