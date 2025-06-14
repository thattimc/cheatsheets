# Loki Cheatsheet

Loki is a horizontally scalable, highly available, multi-tenant log aggregation system inspired by Prometheus. It is designed to be very cost effective and easy to operate, as it does not index the contents of the logs, but rather a set of labels for each log stream.

## Overview

### Key Features
- **Label-based Indexing** - Only indexes labels, not log content
- **Cost Effective** - Minimal storage and compute requirements
- **Multi-tenant** - Supports multiple tenants with isolation
- **Prometheus Integration** - Uses same service discovery and labels
- **Grafana Integration** - Native log exploration in Grafana
- **Push Model** - Agents push logs to Loki
- **LogQL** - Powerful query language for log exploration
- **Horizontal Scaling** - Scales out across multiple instances

### Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Log Sources   │    │     Agents      │    │      Loki       │
│                 │    │                 │    │                 │
│ • Applications  │───►│ • Promtail      │───►│ • Distributor   │
│ • Containers    │    │ • Fluentd       │    │ • Ingester      │
│ • System Logs   │    │ • Fluent Bit    │    │ • Querier       │
│ • Web Servers   │    │ • Vector        │    │ • Query Frontend│
│                 │    │ • Logstash      │    │ • Compactor     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                        │
                                               ┌─────────────────┐
                                               │   Storage       │
                                               │                 │
                                               │ • Object Store  │
                                               │ • File System   │
                                               │ • Index Store   │
                                               └─────────────────┘
                                                        │
                                               ┌─────────────────┐
                                               │    Grafana      │
                                               │                 │
                                               │ • Log Explorer  │
                                               │ • Dashboards    │
                                               │ • Alerting      │
                                               └─────────────────┘
```

### Core Components
- **Distributor** - Receives logs from agents and forwards to ingesters
- **Ingester** - Writes log data to storage and serves recent log queries
- **Querier** - Handles log queries and merges data from ingesters and storage
- **Query Frontend** - Optional service that provides API endpoints and caching
- **Compactor** - Compacts and deduplicates log data in storage
- **Ruler** - Evaluates recording and alerting rules

## Installation

### Docker Installation
```bash
# Simple Loki container
docker run -d \
  --name loki \
  -p 3100:3100 \
  grafana/loki:latest

# With custom configuration
docker run -d \
  --name loki \
  -p 3100:3100 \
  -v /path/to/loki-config.yaml:/etc/loki/local-config.yaml \
  grafana/loki:latest

# With persistent storage
docker run -d \
  --name loki \
  -p 3100:3100 \
  -v loki-data:/loki \
  -v /path/to/loki-config.yaml:/etc/loki/local-config.yaml \
  grafana/loki:latest
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yaml:/etc/loki/local-config.yaml
      - loki-data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    restart: unless-stopped
    networks:
      - logging

  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    volumes:
      - ./promtail-config.yaml:/etc/promtail/config.yml
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    command: -config.file=/etc/promtail/config.yml
    restart: unless-stopped
    networks:
      - logging
    depends_on:
      - loki

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana-datasources.yaml:/etc/grafana/provisioning/datasources/datasources.yaml
    restart: unless-stopped
    networks:
      - logging
    depends_on:
      - loki

volumes:
  loki-data:
  grafana-data:

networks:
  logging:
    driver: bridge
```

### Kubernetes Installation
```bash
# Using Helm
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Loki
helm install loki grafana/loki

# Install Loki Stack (Loki + Promtail + Grafana)
helm install loki-stack grafana/loki-stack \
  --set grafana.enabled=true \
  --set prometheus.enabled=true \
  --set prometheus.alertmanager.persistentVolume.enabled=false \
  --set prometheus.server.persistentVolume.enabled=false

# Get Grafana admin password
kubectl get secret --namespace default loki-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port forward to access Grafana
kubectl port-forward --namespace default service/loki-stack-grafana 3000:80
```

### Binary Installation
```bash
# Download Loki binary
wget https://github.com/grafana/loki/releases/download/v2.9.0/loki-linux-amd64.zip
unzip loki-linux-amd64.zip
sudo mv loki-linux-amd64 /usr/local/bin/loki
sudo chmod +x /usr/local/bin/loki

# Download Promtail binary
wget https://github.com/grafana/loki/releases/download/v2.9.0/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
sudo chmod +x /usr/local/bin/promtail

# Create configuration directory
sudo mkdir -p /etc/loki

# Run Loki
loki -config.file=/etc/loki/loki-config.yaml

# Run Promtail
promtail -config.file=/etc/promtail/config.yml
```

### macOS Installation
```bash
# Using Homebrew
brew install loki
brew install promtail

# Run Loki
loki -config.file=/usr/local/etc/loki-config.yaml

# Run Promtail
promtail -config.file=/usr/local/etc/promtail-config.yaml
```

## Configuration

### Basic Loki Configuration
```yaml
# loki-config.yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096
  log_level: info

common:
  instance_addr: 127.0.0.1
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

query_range:
  results_cache:
    cache:
      embedded_cache:
        enabled: true
        max_size_mb: 100

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093

# By default, Loki will send anonymous, but uniquely-identifiable usage and configuration
# analytics to Grafana Labs. These statistics are sent to https://stats.grafana.org/
#
# Statistics help us better understand how Loki is used, and they show us performance
# levels for most users. This helps us prioritize features and documentation.
# For more information on what's sent, look at
# https://github.com/grafana/loki/blob/main/pkg/usagestats/stats.go
# Refer to the buildReport method to see what goes into a report.
#
# If you would like to disable reporting, uncomment the following lines:
analytics:
  reporting_enabled: false
```

### Production Loki Configuration
```yaml
# loki-production.yaml
auth_enabled: true

server:
  http_listen_port: 3100
  grpc_listen_port: 9096
  log_level: warn
  http_server_read_timeout: 30s
  http_server_write_timeout: 30s

distributor:
  ring:
    kvstore:
      store: consul
      consul:
        host: consul:8500

ingester:
  lifecycler:
    ring:
      kvstore:
        store: consul
        consul:
          host: consul:8500
      replication_factor: 3
    num_tokens: 128
    heartbeat_period: 5s
    observe_period: 10s
    join_after: 10s
    min_ready_duration: 10s
    final_sleep: 30s
  chunk_idle_period: 1h
  chunk_retain_period: 30s
  max_transfer_retries: 0
  wal:
    enabled: true
    dir: /loki/wal
    checkpoint_duration: 5m
    flush_on_shutdown: true

querier:
  query_ingesters_within: 3h

query_frontend:
  max_outstanding_per_tenant: 256
  compress_responses: true
  downstream_url: http://querier:3100

query_range:
  align_queries_with_step: true
  max_retries: 5
  cache_results: true
  results_cache:
    cache:
      redis_cache:
        endpoint: redis:6379
        timeout: 500ms
        expiration: 1h

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h
  ingestion_rate_mb: 10
  ingestion_burst_size_mb: 20
  max_streams_per_user: 0
  max_line_size: 256000
  per_stream_rate_limit: 3MB
  per_stream_rate_limit_burst: 15MB

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: s3
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /loki/boltdb-shipper-active
    cache_location: /loki/boltdb-shipper-cache
    cache_ttl: 24h
    shared_store: s3
  aws:
    s3: s3://loki-bucket/loki
    region: us-west-2
    access_key_id: ${AWS_ACCESS_KEY_ID}
    secret_access_key: ${AWS_SECRET_ACCESS_KEY}

compactor:
  working_directory: /loki/compactor
  shared_store: s3
  compaction_interval: 10m
  retention_enabled: true
  retention_delete_delay: 2h
  retention_delete_worker_count: 150

ruler:
  storage:
    type: local
    local:
      directory: /loki/rules
  rule_path: /loki/rules-temp
  alertmanager_url: http://alertmanager:9093
  ring:
    kvstore:
      store: consul
      consul:
        host: consul:8500
  enable_api: true

table_manager:
  retention_deletes_enabled: true
  retention_period: 168h

analytics:
  reporting_enabled: false
```

### Promtail Configuration
```yaml
# promtail-config.yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  # Local system logs
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*log

  # Application logs
  - job_name: containers
    static_configs:
      - targets:
          - localhost
        labels:
          job: containerlogs
          __path__: /var/lib/docker/containers/*/*log

  # Docker logs with service discovery
  - job_name: docker
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s
    relabel_configs:
      - source_labels: ['__meta_docker_container_name']
        regex: '/(.*)'  
        target_label: 'container'
      - source_labels: ['__meta_docker_container_log_stream']
        target_label: 'stream'
      - source_labels: ['__meta_docker_container_label_com_docker_compose_service']
        target_label: 'service'
    pipeline_stages:
      - docker: {}

  # Journal logs
  - job_name: journal
    journal:
      max_age: 12h
      labels:
        job: systemd-journal
    relabel_configs:
      - source_labels: ['__journal__systemd_unit']
        target_label: 'unit'
      - source_labels: ['__journal__hostname']
        target_label: 'hostname'

  # Kubernetes pods
  - job_name: kubernetes-pods
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels:
          - __meta_kubernetes_pod_controller_name
        regex: ([0-9a-z-.]+?)(-[0-9a-f]{8,10})?
        action: replace
        target_label: __tmp_controller_name
      - source_labels:
          - __meta_kubernetes_pod_label_app_kubernetes_io_name
          - __meta_kubernetes_pod_label_app
          - __tmp_controller_name
          - __meta_kubernetes_pod_name
        regex: ^;*([^;]+)(;.*)?$
        action: replace
        target_label: app
      - source_labels:
          - __meta_kubernetes_pod_label_app_kubernetes_io_instance
          - __meta_kubernetes_pod_label_release
        regex: ^;*([^;]+)(;.*)?$
        action: replace
        target_label: instance
      - source_labels:
          - __meta_kubernetes_pod_label_app_kubernetes_io_component
          - __meta_kubernetes_pod_label_component
        regex: ^;*([^;]+)(;.*)?$
        action: replace
        target_label: component
      - action: replace
        source_labels:
        - __meta_kubernetes_pod_node_name
        target_label: node_name
      - action: replace
        source_labels:
        - __meta_kubernetes_namespace
        target_label: namespace
      - action: replace
        replacement: /var/log/pods/*$1/*.log
        separator: /
        source_labels:
        - __meta_kubernetes_pod_uid
        - __meta_kubernetes_pod_container_name
        target_label: __path__
    pipeline_stages:
      - cri: {}

  # Syslog
  - job_name: syslog
    syslog:
      listen_address: 0.0.0.0:1514
      idle_timeout: 60s
      label_structured_data: yes
      labels:
        job: "syslog"
    relabel_configs:
      - source_labels: ['__syslog_message_hostname']
        target_label: 'host'
    pipeline_stages:
      - regex:
          expression: '^(?P<timestamp>\S+\s+\S+\s+\S+)\s+(?P<host>\S+)\s+(?P<process>\S+)\s+(?P<message>.*)$'
      - timestamp:
          source: timestamp
          format: 'Jan 02 15:04:05'
      - labels:
          process:
```

## LogQL (Log Query Language)

### Basic Syntax
```logql
# Simple log stream selector
{job="varlog"}

# Multiple labels
{job="varlog", level="error"}

# Label matching operators
{job="varlog", level!="info"}     # Not equal
{job=~"var.*"}                    # Regex match
{job!~"test.*"}                   # Regex not match

# All logs from a namespace
{namespace="production"}

# Logs from specific pod
{pod="web-server-123"}
```

### Log Stream Selectors
```logql
# Basic stream selectors
{container="nginx"}
{filename="/var/log/nginx/access.log"}
{service="api", environment="production"}

# Multiple conditions
{job="kubernetes-pods", namespace="default", container="app"}

# Exclude specific streams
{job="varlog"} != "debug"
{namespace!="kube-system"}

# Complex label matching
{container=~"web.*", namespace=~"prod|staging"}
{level=~"error|fatal"}
```

### Line Filter Expressions
```logql
# Contains text
{job="varlog"} |= "error"

# Does not contain text
{job="varlog"} != "debug"

# Regex match
{job="varlog"} |~ "error|warning"

# Regex not match
{job="varlog"} !~ "debug|trace"

# Case insensitive
{job="varlog"} |~ "(?i)error"

# Multiple filters
{job="varlog"} |= "error" |= "database"
{job="varlog"} |= "error" != "connection"
```

### Label Filter Expressions
```logql
# Parse and filter by extracted labels
{job="varlog"} | json | level="error"
{job="varlog"} | logfmt | status_code >= 400
{job="varlog"} | regexp "(?P<level>\\w+)" | level="ERROR"

# Numeric comparisons
{job="nginx"} | json | status_code >= 400
{job="nginx"} | json | response_time > 1.5
{job="nginx"} | json | bytes_sent <= 1024

# String operations
{job="app"} | json | user_id != ""
{job="app"} | json | method =~ "GET|POST"
```

### Parser Expressions
```logql
# JSON parser
{job="app"} | json
{job="app"} | json | level="error"

# Logfmt parser
{job="app"} | logfmt
{job="app"} | logfmt | status_code >= 400

# Regex parser
{job="nginx"} | regexp "(?P<ip>\\S+) .* \"(?P<method>\\w+) (?P<path>\\S+).*\" (?P<status>\\d+)"

# Pattern parser
{job="app"} | pattern "<timestamp> <level> <message>"
{job="nginx"} | pattern "<ip> - - [<timestamp>] \"<method> <path> <protocol>\" <status> <size>"

# Multiple parsers
{job="app"} | json | line_format "{{.timestamp}} {{.level}} {{.message}}"
```

### Aggregation Functions
```logql
# Count logs
count_over_time({job="varlog"}[5m])

# Rate of logs
rate({job="varlog"}[5m])

# Count by level
sum by (level) (count_over_time({job="app"} | json [5m]))

# Error rate
sum(rate({job="app"} |= "error" [5m])) / sum(rate({job="app"}[5m]))

# Top 10 error messages
topk(10, sum by (message) (count_over_time({job="app"} |= "error" [1h])))

# Average response time
avg_over_time({job="nginx"} | json | unwrap response_time [5m])

# 95th percentile
quantile_over_time(0.95, {job="nginx"} | json | unwrap response_time [5m])
```

### Advanced Queries
```logql
# Error rate by service
sum by (service) (
  rate({namespace="production"} |= "error" [5m])
) / 
sum by (service) (
  rate({namespace="production"}[5m])
)

# Log volume by container
sum by (container) (count_over_time({job="kubernetes-pods"}[1h]))

# Failed requests by endpoint
sum by (path) (
  count_over_time(
    {job="nginx"} 
    | json 
    | status_code >= 400 [1h]
  )
)

# Response time percentiles
quantile_over_time(0.50, {job="api"} | json | unwrap duration [5m]) or
quantile_over_time(0.90, {job="api"} | json | unwrap duration [5m]) or
quantile_over_time(0.95, {job="api"} | json | unwrap duration [5m])

# Detecting patterns
{job="app"} 
| json 
| __error__ = "" 
| level = "error" 
| message =~ ".*timeout.*"

# Multi-line log parsing
{job="java-app"} 
| multiline.firstline.regex "^\\d{4}-\\d{2}-\\d{2}"
| json
| level = "ERROR"
```

### Metric Queries
```logql
# Range aggregations
sum(rate({job="app"}[5m]))
avg(avg_over_time({job="app"} | json | unwrap response_time [5m]))
max(max_over_time({job="app"} | json | unwrap cpu_usage [5m]))

# Vector aggregations
sum by (instance) (rate({job="node"}[5m]))
avg by (service) (avg_over_time({job="app"} | json | unwrap latency [5m]))
topk(5, sum by (container) (count_over_time({job="k8s"}[1h])))

# Binary operations
sum(rate({app="frontend"}[5m])) + sum(rate({app="backend"}[5m]))
sum(rate({job="app"} |= "error" [5m])) / sum(rate({job="app"}[5m])) * 100

# Comparison operators
sum by (service) (rate({job="app"}[5m])) > 100
avg by (instance) (avg_over_time({job="node"} | json | unwrap cpu [5m])) > 0.8
```

## Promtail Pipeline Stages

### Basic Pipeline Stages
```yaml
pipeline_stages:
  # Docker log format
  - docker: {}
  
  # CRI log format (Kubernetes)
  - cri: {}
  
  # JSON parsing
  - json:
      expressions:
        level: level
        timestamp: timestamp
        message: message
        user_id: user.id
  
  # Regex parsing
  - regex:
      expression: '^(?P<ip>\S+) .* "(?P<method>\w+) (?P<path>\S+).*" (?P<status>\d+) (?P<size>\d+)'
  
  # Timestamp parsing
  - timestamp:
      source: timestamp
      format: '2006-01-02T15:04:05.000Z'
      location: 'UTC'
```

### Advanced Pipeline Stages
```yaml
pipeline_stages:
  # Multi-line log handling
  - multiline:
      firstline: '^\d{4}-\d{2}-\d{2}'
      max_wait_time: 3s
      max_lines: 1000
  
  # Template stage for log formatting
  - template:
      source: output
      template: '{{ .timestamp }} [{{ .level }}] {{ .message }}'
  
  # Output stage to modify log line
  - output:
      source: formatted_message
  
  # Label extraction
  - labels:
      level:
      method:
      status_code:
  
  # Metrics extraction
  - metrics:
      request_duration:
        type: Histogram
        description: "HTTP request duration"
        source: duration
        config:
          buckets: [0.1, 0.2, 0.5, 1.0, 2.0, 5.0]
      
      http_requests_total:
        type: Counter
        description: "Total HTTP requests"
        config:
          action: inc
  
  # Drop logs based on conditions
  - drop:
      source: level
      value: debug
      older_than: 24h
  
  # Sampling
  - sampling:
      rate: 0.1  # Keep 10% of logs
```

### Example Complete Pipeline
```yaml
scrape_configs:
  - job_name: nginx
    static_configs:
      - targets:
          - localhost
        labels:
          job: nginx
          __path__: /var/log/nginx/access.log
    pipeline_stages:
      # Parse nginx access log
      - regex:
          expression: '^(?P<remote_addr>\S+) - (?P<remote_user>\S+) \[(?P<time_local>[^\]]+)\] "(?P<method>\S+) (?P<request_uri>\S+) (?P<server_protocol>\S+)" (?P<status>\d+) (?P<body_bytes_sent>\d+) "(?P<http_referer>[^"]*)" "(?P<http_user_agent>[^"]*)".* (?P<request_time>\d+\.\d+)'
      
      # Parse timestamp
      - timestamp:
          source: time_local
          format: '02/Jan/2006:15:04:05 -0700'
      
      # Add labels
      - labels:
          method:
          status:
          remote_addr:
      
      # Create metrics
      - metrics:
          nginx_http_requests_total:
            type: Counter
            description: "Total number of HTTP requests"
            config:
              action: inc
          
          nginx_http_request_duration_seconds:
            type: Histogram
            description: "HTTP request duration in seconds"
            source: request_time
            config:
              buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10]
      
      # Drop debug logs in production
      - match:
          selector: '{job="nginx"}'
          stages:
            - drop:
                source: status
                expression: '^[23]\d{2}$'  # Drop 2xx and 3xx status codes
                drop_counter_reason: "success_logs_dropped"
```

## Grafana Integration

### Data Source Configuration
```yaml
# grafana-datasources.yaml
apiVersion: 1

datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: false
    editable: true
    jsonData:
      maxLines: 1000
      derivedFields:
        - datasourceUid: jaeger
          matcherRegex: "traceID=(\\w+)"
          name: TraceID
          url: "$${__value.raw}"
      alertmanager:
        uid: alertmanager
```

### Log Panel Configuration
```json
{
  "type": "logs",
  "title": "Application Logs",
  "targets": [
    {
      "expr": "{job=\"app\"} |= \"error\"",
      "refId": "A"
    }
  ],
  "options": {
    "showTime": true,
    "showLabels": false,
    "showCommonLabels": false,
    "wrapLogMessage": false,
    "prettifyLogMessage": false,
    "enableLogDetails": true,
    "dedupStrategy": "none",
    "sortOrder": "Descending"
  },
  "fieldConfig": {
    "defaults": {
      "custom": {
        "stacking": {
          "mode": "none",
          "group": "A"
        }
      }
    }
  }
}
```

### Log Rate Panel
```json
{
  "type": "timeseries",
  "title": "Log Rate",
  "targets": [
    {
      "expr": "sum(rate({job=\"app\"}[5m]))",
      "legendFormat": "Total Logs/sec",
      "refId": "A"
    },
    {
      "expr": "sum(rate({job=\"app\"} |= \"error\"[5m]))",
      "legendFormat": "Error Logs/sec",
      "refId": "B"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "unit": "logs/sec",
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "linear",
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": false,
        "pointSize": 5,
        "stacking": {
          "mode": "none",
          "group": "A"
        }
      }
    }
  }
}
```

## Alerting

### Recording Rules
```yaml
# loki-rules.yaml
groups:
  - name: loki_rules
    rules:
      - record: job:log_rate5m
        expr: sum(rate({job=~".+"}[5m])) by (job)
      
      - record: job:error_rate5m
        expr: sum(rate({job=~".+"} |= "error"[5m])) by (job)
      
      - record: job:error_ratio5m
        expr: |
          (
            sum(rate({job=~".+"} |= "error"[5m])) by (job)
          /
            sum(rate({job=~".+"}[5m])) by (job)
          )
```

### Alerting Rules
```yaml
# loki-alerts.yaml
groups:
  - name: loki_alerts
    rules:
      - alert: HighLogErrorRate
        expr: |
          (
            sum(rate({job=~".+"} |= "error"[5m])) by (job)
          /
            sum(rate({job=~".+"}[5m])) by (job)
          ) > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate in logs"
          description: "Job {{ $labels.job }} has error rate of {{ $value | humanizePercentage }}"
      
      - alert: LogVolumeSpike
        expr: |
          (
            sum(rate({job=~".+"}[5m])) by (job)
          >
            sum(rate({job=~".+"}[1h])) by (job) * 2
          )
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Log volume spike detected"
          description: "Job {{ $labels.job }} log volume is {{ $value }} logs/sec, which is unusually high"
      
      - alert: NoLogsReceived
        expr: |
          (
            sum(rate({job=~".+"}[15m])) by (job) == 0
          )
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "No logs received"
          description: "No logs received from job {{ $labels.job }} for 15 minutes"
      
      - alert: HighMemoryUsage
        expr: |
          (
            sum(count_over_time({job="app"} | json | message =~ ".*OutOfMemoryError.*"[5m]))
          > 0
          )
        for: 0s
        labels:
          severity: critical
        annotations:
          summary: "Application out of memory errors detected"
          description: "OutOfMemoryError detected in application logs"
```

## Storage and Retention

### Storage Configuration
```yaml
# File system storage
storage_config:
  boltdb_shipper:
    active_index_directory: /loki/boltdb-shipper-active
    cache_location: /loki/boltdb-shipper-cache
    cache_ttl: 24h
    shared_store: filesystem
  filesystem:
    directory: /loki/chunks

# AWS S3 storage
storage_config:
  boltdb_shipper:
    active_index_directory: /loki/boltdb-shipper-active
    cache_location: /loki/boltdb-shipper-cache
    cache_ttl: 24h
    shared_store: s3
  aws:
    s3: s3://my-loki-bucket/loki
    region: us-west-2
    access_key_id: ${AWS_ACCESS_KEY_ID}
    secret_access_key: ${AWS_SECRET_ACCESS_KEY}
    sse_encryption: true
    kms_key_id: alias/loki-kms-key

# Google Cloud Storage
storage_config:
  boltdb_shipper:
    active_index_directory: /loki/boltdb-shipper-active
    cache_location: /loki/boltdb-shipper-cache
    shared_store: gcs
  gcs:
    bucket_name: my-loki-bucket
    chunk_buffer_size: 0
    request_timeout: 0s
    enable_http2: true
```

### Retention Configuration
```yaml
# Table manager retention
table_manager:
  retention_deletes_enabled: true
  retention_period: 168h  # 7 days
  poll_interval: 10m
  creation_grace_period: 3h
  index_tables_provisioning:
    enable_ondemand_throughput_mode: false
    provisioned_read_throughput: 1000
    provisioned_write_throughput: 3000

# Compactor retention
compactor:
  working_directory: /loki/compactor
  shared_store: s3
  compaction_interval: 10m
  retention_enabled: true
  retention_delete_delay: 2h
  retention_delete_worker_count: 150
  delete_request_cancel_period: 24h

# Per-tenant retention
limits_config:
  retention_period: 720h  # 30 days
  retention_stream:
    - selector: '{environment="development"}'
      priority: 1
      period: 24h
    - selector: '{environment="staging"}'
      priority: 1
      period: 168h  # 7 days
    - selector: '{environment="production"}'
      priority: 1
      period: 2160h  # 90 days
```

## Performance Tuning

### Ingester Configuration
```yaml
ingester:
  # WAL configuration
  wal:
    enabled: true
    dir: /loki/wal
    checkpoint_duration: 5m
    flush_on_shutdown: true
    replay_memory_ceiling: 1GB
  
  # Chunk configuration
  chunk_idle_period: 1h
  chunk_retain_period: 30s
  chunk_target_size: 1572864  # 1.5MB
  chunk_encoding: snappy
  
  # Lifecycler configuration
  lifecycler:
    heartbeat_period: 5s
    join_after: 10s
    min_ready_duration: 10s
    final_sleep: 30s
    num_tokens: 128
  
  # Transfer configuration
  max_transfer_retries: 0
  concurrent_flushes: 16
  flush_check_period: 30s
  flush_op_timeout: 10s
```

### Query Performance
```yaml
querier:
  # Query performance
  query_timeout: 300s
  query_ingesters_within: 3h
  max_concurrent: 16
  
  # Engine configuration
  engine:
    timeout: 300s
    max_look_back_period: 30s

query_range:
  # Results caching
  cache_results: true
  max_retries: 5
  parallelise_shardable_queries: true
  
  # Query splitting
  split_queries_by_interval: 15m
  align_queries_with_step: true
  
  # Results cache
  results_cache:
    cache:
      redis_cache:
        endpoint: redis:6379
        timeout: 500ms
        expiration: 1h
        max_size_mb: 100

query_frontend:
  # Query queuing
  max_outstanding_per_tenant: 256
  
  # Query sharding
  split_queries_by_interval: 15m
  parallelise_shardable_queries: true
  
  # Compression
  compress_responses: true
  
  # Downstream URL
  downstream_url: http://querier:3100
```

### Limits Configuration
```yaml
limits_config:
  # Ingestion limits
  ingestion_rate_mb: 10
  ingestion_burst_size_mb: 20
  max_line_size: 256000
  max_line_size_truncate: true
  
  # Stream limits
  max_streams_per_user: 0
  max_global_streams_per_user: 5000
  per_stream_rate_limit: 3MB
  per_stream_rate_limit_burst: 15MB
  
  # Query limits
  max_query_length: 721h
  max_query_parallelism: 32
  max_query_series: 1000
  max_concurrent_tail_requests: 10
  
  # Cardinality limits
  max_label_name_length: 1024
  max_label_value_length: 4096
  max_label_names_per_series: 30
  
  # Deletion
  deletion_mode: filter-and-delete
  retention_period: 744h  # 31 days
```

## Monitoring Loki

### Key Metrics
```promql
# Ingestion metrics
loki_ingester_chunks_flushed_total
loki_ingester_chunk_age_seconds
loki_ingester_memory_chunks
loki_ingester_received_chunks

# Query metrics
loki_query_duration_seconds
loki_query_frontend_queries_total
loki_querier_series_per_query

# Storage metrics
loki_chunk_store_index_entries_per_chunk
loki_chunk_store_chunks_per_query
loki_boltdb_shipper_files_downloaded_total

# Distributor metrics
loki_distributor_received_samples_total
loki_distributor_bytes_received_total
loki_distributor_lines_received_total

# Compactor metrics
loki_compactor_blocks_marked_for_deletion_total
loki_compactor_delete_requests_processed_total
```

### Health Checks
```bash
# Loki readiness
curl http://localhost:3100/ready

# Loki metrics
curl http://localhost:3100/metrics

# Loki configuration
curl http://localhost:3100/config

# Loki services
curl http://localhost:3100/services

# Ring status
curl http://localhost:3100/ring

# Ruler status
curl http://localhost:3100/ruler/ring

# Compactor status
curl http://localhost:3100/compactor/ring
```

## API Reference

### Push API
```bash
# Push logs to Loki
curl -v -H "Content-Type: application/json" -XPOST \
  "http://localhost:3100/loki/api/v1/push" \
  --data-raw '{
    "streams": [
      {
        "stream": {
          "job": "test",
          "level": "info"
        },
        "values": [
          [ "'$(date +%s%N)'", "test log message" ]
        ]
      }
    ]
  }'

# Push with labels
curl -H "Content-Type: application/json" -XPOST \
  "http://localhost:3100/loki/api/v1/push" \
  --data-raw '{
    "streams": [
      {
        "stream": {
          "service": "api",
          "environment": "production",
          "level": "error"
        },
        "values": [
          [ "'$(date +%s%N)'", "Database connection failed" ]
        ]
      }
    ]
  }'
```

### Query API
```bash
# Query logs
curl -G -s "http://localhost:3100/loki/api/v1/query" \
  --data-urlencode 'query={job="test"}' \
  --data-urlencode 'limit=100'

# Range query
curl -G -s "http://localhost:3100/loki/api/v1/query_range" \
  --data-urlencode 'query={job="test"}' \
  --data-urlencode 'start=2023-01-01T00:00:00Z' \
  --data-urlencode 'end=2023-01-01T01:00:00Z' \
  --data-urlencode 'step=5m'

# Label names
curl -G -s "http://localhost:3100/loki/api/v1/labels"

# Label values
curl -G -s "http://localhost:3100/loki/api/v1/label/job/values"

# Series
curl -G -s "http://localhost:3100/loki/api/v1/series" \
  --data-urlencode 'match[]={job="test"}'

# Tail logs (WebSocket)
wscat -c "ws://localhost:3100/loki/api/v1/tail?query={job=\"test\"}"
```

### Rules API
```bash
# List rule groups
curl -s "http://localhost:3100/loki/api/v1/rules"

# Get specific rule group
curl -s "http://localhost:3100/loki/api/v1/rules/namespace/group"

# Set rule group
curl -X POST "http://localhost:3100/loki/api/v1/rules/namespace" \
  -H "Content-Type: application/yaml" \
  --data-binary @rules.yaml

# Delete rule group
curl -X DELETE "http://localhost:3100/loki/api/v1/rules/namespace/group"
```

## Troubleshooting

### Common Issues
```bash
# Check Loki logs
docker logs loki
kubectl logs -f deployment/loki

# Check Promtail logs
docker logs promtail
kubectl logs -f daemonset/promtail

# Verify connectivity
curl http://localhost:3100/ready
curl http://localhost:3100/metrics

# Check configuration
loki -config.file=/etc/loki/config.yaml -verify-config
promtail -config.file=/etc/promtail/config.yml -dry-run

# Test log pushing
curl -v -H "Content-Type: application/json" -XPOST \
  "http://localhost:3100/loki/api/v1/push" \
  --data-raw '{"streams":[{"stream":{"job":"test"},"values":[["'$(date +%s%N)'","test message"]]}]}'

# Check storage permissions
ls -la /loki/
chown -R loki:loki /loki/
```

### Performance Issues
```bash
# Check ingestion rate
curl -s http://localhost:3100/metrics | grep loki_distributor_bytes_received_total

# Check memory usage
docker stats loki
kubectl top pod -l app=loki

# Check disk usage
df -h /loki/
du -sh /loki/*

# Monitor query performance
curl -s http://localhost:3100/metrics | grep loki_query_duration_seconds

# Check chunk flushing
curl -s http://localhost:3100/metrics | grep loki_ingester_chunks_flushed_total
```

### Debug Configuration
```yaml
server:
  log_level: debug
  log_format: json

# Enable query debugging
querier:
  query_timeout: 300s
  engine:
    timeout: 300s
    max_look_back_period: 30s

# Enable detailed metrics
analytics:
  reporting_enabled: false

# Add extra labels for debugging
limits_config:
  enforce_metric_name: false
  reject_old_samples: false
```

## Best Practices

### Label Design
```yaml
# Good label practices
labels:
  service: "api"          # Low cardinality
  environment: "prod"     # Low cardinality  
  level: "error"          # Low cardinality
  cluster: "us-west"      # Low cardinality

# Avoid high cardinality labels
# Bad examples:
# user_id: "12345"        # High cardinality
# request_id: "abc123"    # High cardinality
# timestamp: "2023..."    # High cardinality
# ip_address: "1.2.3.4"  # High cardinality
```

### Query Optimization
```logql
# Use specific time ranges
{job="app"}[1h]  # Good
{job="app"}[24h] # Avoid if possible

# Filter early in the query
{job="app"} |= "error" | json | level="ERROR"  # Good
{job="app"} | json | level="ERROR" |= "error" # Less efficient

# Use appropriate aggregations
sum by (service) (rate({job="app"}[5m]))  # Good for dashboards
count_over_time({job="app"}[5m])          # Good for counting

# Avoid expensive operations
{job="app"} | json | line_format "{{.message}}"  # Can be expensive
```

### Storage Optimization
```yaml
# Chunk configuration
ingester:
  chunk_target_size: 1572864  # 1.5MB chunks
  chunk_encoding: snappy       # Good compression ratio
  chunk_idle_period: 1h        # Balance between memory and latency

# Retention strategy
limits_config:
  retention_period: 168h       # 7 days default
  retention_stream:
    - selector: '{level="debug"}'
      period: 24h               # Short retention for debug logs
    - selector: '{level="error"}'
      period: 720h              # Long retention for errors
```

---

*For more detailed information, visit the [official Loki documentation](https://grafana.com/docs/loki/)*

