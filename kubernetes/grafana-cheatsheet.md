# Grafana Cheatsheet

Grafana is an open-source platform for monitoring and observability that allows you to
query, visualize, alert on, and understand your metrics no matter where they are stored.

## Overview

### Key Features

- **Data Source Agnostic** - Connect to 60+ data sources
- **Rich Visualizations** - Graphs, tables, heatmaps, and more
- **Dashboard Creation** - Interactive and dynamic dashboards
- **Alerting** - Unified alerting system with multiple notification channels
- **Annotations** - Rich context and metadata overlay
- **Variables** - Template dashboards for reusability
- **Plugins** - Extensible with custom panels and data sources
- **Teams & Organizations** - Multi-tenancy and access control
- **API** - Comprehensive REST API for automation

### Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Data Sources  │    │     Grafana     │    │     Users       │
│                 │    │     Server      │    │                 │
│ • Prometheus    │◄───┤                 ├───►│ • Dashboards    │
│ • InfluxDB      │    │ • Query Engine  │    │ • Alerts        │
│ • Elasticsearch │    │ • Visualization │    │ • Admin         │
│ • MySQL         │    │ • Alerting      │    │                 │
│ • CloudWatch    │    │ • User Mgmt     │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                       ┌─────────────────┐
                       │  Notification   │
                       │   Channels      │
                       │                 │
                       │ • Slack         │
                       │ • Email         │
                       │ • PagerDuty     │
                       │ • Webhook       │
                       └─────────────────┘
```

## Installation

### Docker Installation

```bash
# Basic Grafana container
docker run -d \
  --name grafana \
  -p 3000:3000 \
  grafana/grafana-oss

# With persistent storage
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -v grafana-storage:/var/lib/grafana \
  grafana/grafana-oss

# With environment variables
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -e GF_SECURITY_ADMIN_PASSWORD=admin123 \
  -e GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-piechart-panel \
  -v grafana-storage:/var/lib/grafana \
  grafana/grafana-oss

# Enterprise version
docker run -d \
  --name grafana \
  -p 3000:3000 \
  grafana/grafana-enterprise
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  grafana:
    image: grafana/grafana-oss:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-piechart-panel
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana.ini:/etc/grafana/grafana.ini
      - ./provisioning:/etc/grafana/provisioning
    restart: unless-stopped
    networks:
      - monitoring

volumes:
  grafana-data:

networks:
  monitoring:
    driver: bridge
```

### Kubernetes Installation

```bash
# Using Helm
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install with default values
helm install grafana grafana/grafana

# Install with custom values
helm install grafana grafana/grafana \
  --set persistence.enabled=true \
  --set persistence.size=10Gi \
  --set adminPassword=admin123 \
  --set service.type=LoadBalancer

# Get admin password
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port forward to access
kubectl port-forward service/grafana 3000:80
```

### Binary Installation

```bash
# Ubuntu/Debian
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana

# CentOS/RHEL
wget https://dl.grafana.com/oss/release/grafana-10.2.0-1.x86_64.rpm
sudo yum install grafana-10.2.0-1.x86_64.rpm

# Start Grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Check status
sudo systemctl status grafana-server
```

### macOS Installation

```bash
# Using Homebrew
brew install grafana

# Start Grafana
brew services start grafana

# Or run directly
grafana-server --config=/usr/local/etc/grafana/grafana.ini

# Access Grafana at http://localhost:3000
# Default credentials: admin/admin
```

## Configuration

### Basic Configuration (grafana.ini)

```ini
# /etc/grafana/grafana.ini

[server]
http_port = 3000
protocol = http
domain = localhost
root_url = http://localhost:3000
serve_from_sub_path = false

[database]
type = sqlite3
host = 127.0.0.1:3306
name = grafana
user = root
password =
url =
ssl_mode = disable
path = grafana.db

[security]
admin_user = admin
admin_password = admin
secret_key = SW2YcwTIb9zpOOhoPsMm
login_remember_days = 7
cookie_username = grafana_user
cookie_remember_name = grafana_remember

[users]
allow_sign_up = false
allow_org_create = true
auto_assign_org = true
auto_assign_org_id = 1
auto_assign_org_role = Viewer
verify_email_enabled = false
login_hint = email or username
password_hint = password

[auth]
login_cookie_name = grafana_sess
login_maximum_inactive_lifetime_duration = 7d
login_maximum_lifetime_duration = 30d
token_rotation_interval_minutes = 10

[auth.anonymous]
enabled = false
org_name = Main Org.
org_role = Viewer
hide_version = false

[smtp]
enabled = false
host = localhost:587
user =
password =
cert_file =
key_file =
skip_verify = false
from_address = admin@grafana.localhost
from_name = Grafana
ehlo_identity = dashboard.example.com
startTLS_policy =

[log]
mode = console file
level = info
filters =

[log.console]
level =
format = console

[log.file]
level =
format = text
log_rotate = true
max_lines = 1000000
max_size_shift = 28
daily_rotate = true
max_days = 7

[alerting]
execute_alerts = true
error_or_timeout = alerting
nodata_or_nullvalues = no_data
concurrent_render_limit = 5
evaluation_timeout_seconds = 30
notification_timeout_seconds = 30
max_attempts = 3
min_interval_seconds = 1

[unified_alerting]
enabled = true
disabled_orgs =
min_interval = 10s
max_interval = 60s

[feature_toggles]
enable = publicDashboards
```

### Environment Variables

```bash
# Common environment variables
export GF_SECURITY_ADMIN_PASSWORD=admin123
export GF_USERS_ALLOW_SIGN_UP=false
export GF_USERS_AUTO_ASSIGN_ORG_ROLE=Editor
export GF_AUTH_ANONYMOUS_ENABLED=true
export GF_AUTH_ANONYMOUS_ORG_ROLE=Viewer
export GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-piechart-panel
export GF_DATABASE_TYPE=mysql
export GF_DATABASE_HOST=mysql:3306
export GF_DATABASE_NAME=grafana
export GF_DATABASE_USER=grafana
export GF_DATABASE_PASSWORD=password
export GF_SMTP_ENABLED=true
export GF_SMTP_HOST=smtp.gmail.com:587
export GF_SMTP_USER=your-email@gmail.com
export GF_SMTP_PASSWORD=your-password
export GF_SMTP_FROM_ADDRESS=your-email@gmail.com
```

## Data Sources

### Prometheus Data Source

```yaml
# prometheus-datasource.yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
    jsonData:
      httpMethod: POST
      prometheusType: Prometheus
      prometheusVersion: 2.40.0
      cacheLevel: 'High'
      disableMetricsLookup: false
      incrementalQueryOverlapWindow: 10m
      queryTimeout: 60s
      timeInterval: 15s
```

### InfluxDB Data Source

```yaml
# influxdb-datasource.yaml
apiVersion: 1
datasources:
  - name: InfluxDB
    type: influxdb
    access: proxy
    url: http://influxdb:8086
    database: mydb
    user: admin
    secureJsonData:
      password: password
    jsonData:
      httpMode: GET
      keepCookies: []
```

### MySQL Data Source

```yaml
# mysql-datasource.yaml
apiVersion: 1
datasources:
  - name: MySQL
    type: mysql
    url: mysql:3306
    database: myapp
    user: grafana
    secureJsonData:
      password: password
    jsonData:
      maxOpenConns: 100
      maxIdleConns: 100
      maxLifetime: 14400
```

### Elasticsearch Data Source

```yaml
# elasticsearch-datasource.yaml
apiVersion: 1
datasources:
  - name: Elasticsearch
    type: elasticsearch
    access: proxy
    url: http://elasticsearch:9200
    database: "[logs-]YYYY.MM.DD"
    jsonData:
      interval: Daily
      timeField: "@timestamp"
      esVersion: "7.10.0"
      maxConcurrentShardRequests: 5
      logMessageField: message
      logLevelField: level
```

### CloudWatch Data Source

```yaml
# cloudwatch-datasource.yaml
apiVersion: 1
datasources:
  - name: CloudWatch
    type: cloudwatch
    jsonData:
      authType: keys
      defaultRegion: us-east-1
    secureJsonData:
      accessKey: YOUR_ACCESS_KEY
      secretKey: YOUR_SECRET_KEY
```

## Dashboard Creation

### Dashboard JSON Structure

```json
{
  "dashboard": {
    "id": null,
    "title": "My Dashboard",
    "description": "Dashboard description",
    "tags": ["monitoring", "production"],
    "timezone": "browser",
    "refresh": "30s",
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "timepicker": {
      "refresh_intervals": ["5s", "10s", "30s", "1m", "5m", "15m", "30m", "1h", "2h", "1d"],
      "time_options": ["5m", "15m", "1h", "6h", "12h", "24h", "2d", "7d", "30d"]
    },
    "panels": [],
    "templating": {
      "list": []
    },
    "annotations": {
      "list": []
    },
    "schemaVersion": 37,
    "version": 1,
    "links": []
  },
  "folderId": null,
  "overwrite": false
}
```

### Basic Panel Configuration

```json
{
  "id": 1,
  "title": "CPU Usage",
  "type": "timeseries",
  "gridPos": {
    "h": 8,
    "w": 12,
    "x": 0,
    "y": 0
  },
  "targets": [
    {
      "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
      "legendFormat": "{{instance}}",
      "refId": "A"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "unit": "percent",
      "min": 0,
      "max": 100,
      "thresholds": {
        "steps": [
          {"color": "green", "value": null},
          {"color": "yellow", "value": 70},
          {"color": "red", "value": 90}
        ]
      }
    }
  },
  "options": {
    "legend": {
      "displayMode": "table",
      "placement": "bottom",
      "calcs": ["last", "max"]
    }
  }
}
```

## Panel Types and Configurations

### Time Series Panel

```json
{
  "type": "timeseries",
  "title": "Request Rate",
  "targets": [
    {
      "expr": "sum(rate(http_requests_total[5m])) by (job)",
      "legendFormat": "{{job}}"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "unit": "reqps",
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
        },
        "axisPlacement": "auto",
        "scaleDistribution": {
          "type": "linear"
        }
      }
    }
  }
}
```

### Stat Panel

```json
{
  "type": "stat",
  "title": "Total Requests",
  "targets": [
    {
      "expr": "sum(http_requests_total)",
      "instant": true
    }
  ],
  "fieldConfig": {
    "defaults": {
      "unit": "short",
      "mappings": [],
      "thresholds": {
        "steps": [
          {"color": "green", "value": null},
          {"color": "red", "value": 1000000}
        ]
      }
    }
  },
  "options": {
    "reduceOptions": {
      "values": false,
      "calcs": ["lastNotNull"],
      "fields": ""
    },
    "orientation": "auto",
    "textMode": "auto",
    "colorMode": "background",
    "graphMode": "none",
    "justifyMode": "auto"
  }
}
```

### Gauge Panel

```json
{
  "type": "gauge",
  "title": "Memory Usage",
  "targets": [
    {
      "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "unit": "percent",
      "min": 0,
      "max": 100,
      "thresholds": {
        "steps": [
          {"color": "green", "value": null},
          {"color": "yellow", "value": 70},
          {"color": "red", "value": 90}
        ]
      }
    }
  },
  "options": {
    "showThresholdLabels": false,
    "showThresholdMarkers": true
  }
}
```

### Table Panel

```json
{
  "type": "table",
  "title": "Service Status",
  "targets": [
    {
      "expr": "up",
      "format": "table",
      "instant": true
    }
  ],
  "fieldConfig": {
    "defaults": {
      "custom": {
        "align": "auto",
        "displayMode": "auto",
        "filterable": true
      },
      "mappings": [
        {
          "options": {
            "0": {"text": "Down", "color": "red"},
            "1": {"text": "Up", "color": "green"}
          },
          "type": "value"
        }
      ]
    },
    "overrides": [
      {
        "matcher": {"id": "byName", "options": "Value"},
        "properties": [
          {"id": "displayName", "value": "Status"},
          {"id": "custom.width", "value": 80}
        ]
      }
    ]
  }
}
```

### Heatmap Panel

```json
{
  "type": "heatmap",
  "title": "Response Time Distribution",
  "targets": [
    {
      "expr": "sum(rate(http_request_duration_seconds_bucket[5m])) by (le)",
      "format": "heatmap",
      "legendFormat": "{{le}}"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "custom": {
        "hideFrom": {
          "legend": false,
          "tooltip": false,
          "vis": false
        },
        "scaleDistribution": {
          "type": "linear"
        }
      }
    }
  },
  "options": {
    "calculate": false,
    "cellGap": 1,
    "cellValues": {},
    "color": {
      "exponent": 0.5,
      "fill": "dark-orange",
      "mode": "spectrum",
      "reverse": false,
      "scale": "exponential",
      "scheme": "Oranges",
      "steps": 64
    },
    "exemplars": {
      "color": "rgba(255,0,255,0.7)"
    },
    "filterValues": {
      "le": 1e-9
    },
    "legend": {
      "show": false
    },
    "rowsFrame": {
      "layout": "auto"
    },
    "tooltip": {
      "show": true,
      "yHistogram": false
    },
    "yAxis": {
      "axisPlacement": "left",
      "reverse": false,
      "unit": "s"
    }
  }
}
```

## Variables and Templating

### Query Variable

```json
{
  "name": "instance",
  "type": "query",
  "label": "Instance",
  "description": "Select instance",
  "query": "label_values(up, instance)",
  "datasource": {
    "type": "prometheus",
    "uid": "prometheus"
  },
  "refresh": 1,
  "sort": 1,
  "multi": true,
  "includeAll": true,
  "allValue": ".*",
  "current": {
    "selected": false,
    "text": "All",
    "value": "$__all"
  },
  "options": [],
  "regex": "",
  "skipUrlSync": false
}
```

### Custom Variable

```json
{
  "name": "environment",
  "type": "custom",
  "label": "Environment",
  "description": "Select environment",
  "multi": false,
  "includeAll": false,
  "current": {
    "selected": false,
    "text": "production",
    "value": "production"
  },
  "options": [
    {"text": "Production", "value": "production", "selected": true},
    {"text": "Staging", "value": "staging", "selected": false},
    {"text": "Development", "value": "development", "selected": false}
  ],
  "query": "production,staging,development",
  "skipUrlSync": false
}
```

### Datasource Variable

```json
{
  "name": "datasource",
  "type": "datasource",
  "label": "Data Source",
  "description": "Select data source",
  "query": "prometheus",
  "current": {
    "selected": false,
    "text": "Prometheus",
    "value": "prometheus"
  },
  "options": [],
  "refresh": 1,
  "regex": "",
  "skipUrlSync": false
}
```

### Interval Variable

```json
{
  "name": "interval",
  "type": "interval",
  "label": "Interval",
  "description": "Select time interval",
  "query": "1m,5m,10m,30m,1h,6h,12h,1d,7d,14d,30d",
  "current": {
    "selected": false,
    "text": "5m",
    "value": "5m"
  },
  "options": [
    {"text": "1m", "value": "1m", "selected": false},
    {"text": "5m", "value": "5m", "selected": true},
    {"text": "10m", "value": "10m", "selected": false}
  ],
  "auto": false,
  "auto_count": 30,
  "auto_min": "10s",
  "refresh": 2,
  "skipUrlSync": false
}
```

## Alerting

### Alert Rule Configuration

```json
{
  "uid": "alert-rule-1",
  "title": "High CPU Usage",
  "condition": "B",
  "data": [
    {
      "refId": "A",
      "queryType": "",
      "relativeTimeRange": {
        "from": 600,
        "to": 0
      },
      "datasourceUid": "prometheus",
      "model": {
        "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
        "interval": "",
        "refId": "A"
      }
    },
    {
      "refId": "B",
      "queryType": "",
      "relativeTimeRange": {
        "from": 0,
        "to": 0
      },
      "datasourceUid": "-100",
      "model": {
        "conditions": [
          {
            "evaluator": {
              "params": [80],
              "type": "gt"
            },
            "operator": {
              "type": "and"
            },
            "query": {
              "params": ["A"]
            },
            "reducer": {
              "params": [],
              "type": "last"
            },
            "type": "query"
          }
        ],
        "datasource": {
          "type": "__expr__",
          "uid": "-100"
        },
        "expression": "A",
        "hide": false,
        "intervalMs": 1000,
        "maxDataPoints": 43200,
        "reducer": "last",
        "refId": "B",
        "type": "reduce"
      }
    }
  ],
  "intervalSeconds": 60,
  "noDataState": "NoData",
  "execErrState": "Alerting",
  "for": "5m",
  "annotations": {
    "description": "CPU usage is above 80% for more than 5 minutes on instance {{ $labels.instance }}",
    "runbook_url": "https://runbooks.example.com/high-cpu",
    "summary": "High CPU usage detected"
  },
  "labels": {
    "severity": "warning",
    "team": "infrastructure"
  }
}
```

### Notification Policy

```json
{
  "receiver": "default-receiver",
  "group_by": ["alertname", "cluster", "service"],
  "group_wait": "10s",
  "group_interval": "10s",
  "repeat_interval": "1h",
  "routes": [
    {
      "receiver": "critical-alerts",
      "group_wait": "0s",
      "matchers": [
        {
          "name": "severity",
          "value": "critical",
          "isRegex": false,
          "isEqual": true
        }
      ]
    },
    {
      "receiver": "team-alerts",
      "matchers": [
        {
          "name": "team",
          "value": "backend",
          "isRegex": false,
          "isEqual": true
        }
      ]
    }
  ]
}
```

### Contact Points

```json
{
  "name": "slack-alerts",
  "type": "slack",
  "settings": {
    "url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
    "username": "Grafana",
    "channel": "#alerts",
    "iconEmoji": ":exclamation:",
    "iconUrl": "",
    "title": "{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}",
    "text": "{{ range .Alerts }}{{ .Annotations.description }}{{ end }}",
    "color": "{{ if eq .Status \"firing\" }}danger{{ else }}good{{ end }}"
  }
}
```

## Provisioning

### Dashboard Provisioning

```yaml
# provisioning/dashboards/dashboard.yaml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards

  - name: 'infrastructure'
    orgId: 1
    folder: 'Infrastructure'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards/infrastructure

  - name: 'applications'
    orgId: 1
    folder: 'Applications'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards/applications
```

### Data Source Provisioning

```yaml
# provisioning/datasources/datasource.yaml
apiVersion: 1

deleteDatasources:
  - name: Old Prometheus
    orgId: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    orgId: 1
    url: http://prometheus:9090
    password:
    user:
    database:
    basicAuth: false
    basicAuthUser:
    basicAuthPassword:
    withCredentials:
    isDefault: true
    jsonData:
      graphiteVersion: "1.1"
      tlsAuth: false
      tlsAuthWithCACert: false
      httpMethod: POST
      prometheusType: Prometheus
      prometheusVersion: 2.40.0
      cacheLevel: 'High'
      disableMetricsLookup: false
      incrementalQueryOverlapWindow: 10m
      queryTimeout: 60s
      timeInterval: 15s
    secureJsonData:
      tlsCACert: "..."
      tlsClientCert: "..."
      tlsClientKey: "..."
    version: 1
    editable: true

  - name: Loki
    type: loki
    access: proxy
    orgId: 1
    url: http://loki:3100
    isDefault: false
    version: 1
    editable: true
    jsonData:
      maxLines: 1000
      derivedFields:
        - datasourceUid: jaeger
          matcherRegex: "traceID=(\\w+)"
          name: TraceID
          url: "$${__value.raw}"
```

### Alert Provisioning

```yaml
# provisioning/alerting/alerts.yaml
groups:
  - name: instance-alerts
    orgId: 1
    folder: alerts
    interval: 1m
    rules:
      - uid: instance-down
        title: Instance Down
        condition: B
        data:
          - refId: A
            queryType: ''
            relativeTimeRange:
              from: 600
              to: 0
            datasourceUid: prometheus
            model:
              expr: up
              interval: ''
              refId: A
          - refId: B
            queryType: ''
            relativeTimeRange:
              from: 0
              to: 0
            datasourceUid: __expr__
            model:
              conditions:
                - evaluator:
                    params: [1]
                    type: lt
                  operator:
                    type: and
                  query:
                    params: [A]
                  reducer:
                    params: []
                    type: last
                  type: query
              datasource:
                type: __expr__
                uid: __expr__
              expression: A
              intervalMs: 1000
              maxDataPoints: 43200
              reducer: last
              refId: B
              type: reduce
        noDataState: NoData
        execErrState: Alerting
        for: 5m
        annotations:
          description: 'Instance {{ $labels.instance }} is down'
          summary: 'Instance down'
        labels:
          severity: critical
```

## Grafana API

### Authentication

```bash
# API Key authentication
API_KEY="your-api-key"
HEADER="Authorization: Bearer $API_KEY"

# Basic authentication
USER="admin"
PASS="admin"
AUTH="-u $USER:$PASS"

# Service account token
SA_TOKEN="your-service-account-token"
HEADER="Authorization: Bearer $SA_TOKEN"
```

### Dashboard API

```bash
# Get all dashboards
curl -H "$HEADER" "http://localhost:3000/api/search?type=dash-db"

# Get dashboard by UID
curl -H "$HEADER" "http://localhost:3000/api/dashboards/uid/dashboard-uid"

# Create/Update dashboard
curl -X POST -H "$HEADER" -H "Content-Type: application/json" \
  "http://localhost:3000/api/dashboards/db" \
  -d @dashboard.json

# Delete dashboard
curl -X DELETE -H "$HEADER" "http://localhost:3000/api/dashboards/uid/dashboard-uid"

# Get dashboard tags
curl -H "$HEADER" "http://localhost:3000/api/dashboards/tags"

# Search dashboards
curl -H "$HEADER" "http://localhost:3000/api/search?query=cpu&tag=monitoring"

# Get dashboard permissions
curl -H "$HEADER" "http://localhost:3000/api/dashboards/uid/dashboard-uid/permissions"

# Set dashboard permissions
curl -X POST -H "$HEADER" -H "Content-Type: application/json" \
  "http://localhost:3000/api/dashboards/uid/dashboard-uid/permissions" \
  -d '{
    "items": [
      {"role": "Viewer", "permission": 1},
      {"role": "Editor", "permission": 2}
    ]
  }'
```

### Data Source API

```bash
# Get all data sources
curl -H "$HEADER" "http://localhost:3000/api/datasources"

# Get data source by ID
curl -H "$HEADER" "http://localhost:3000/api/datasources/1"

# Get data source by name
curl -H "$HEADER" "http://localhost:3000/api/datasources/name/Prometheus"

# Create data source
curl -X POST -H "$HEADER" -H "Content-Type: application/json" \
  "http://localhost:3000/api/datasources" \
  -d '{
    "name": "Test Prometheus",
    "type": "prometheus",
    "url": "http://prometheus:9090",
    "access": "proxy",
    "isDefault": false
  }'

# Update data source
curl -X PUT -H "$HEADER" -H "Content-Type: application/json" \
  "http://localhost:3000/api/datasources/1" \
  -d @datasource.json

# Delete data source
curl -X DELETE -H "$HEADER" "http://localhost:3000/api/datasources/1"

# Test data source
curl -X POST -H "$HEADER" "http://localhost:3000/api/datasources/1/health"
```

### User and Organization API

```bash
# Get current user
curl -H "$HEADER" "http://localhost:3000/api/user"

# Get all users (admin only)
curl -H "$HEADER" "http://localhost:3000/api/users"

# Create user
curl -X POST -H "$HEADER" -H "Content-Type: application/json" \
  "http://localhost:3000/api/admin/users" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "login": "john",
    "password": "password",
    "OrgId": 1
  }'

# Get organizations
curl -H "$HEADER" "http://localhost:3000/api/orgs"

# Get current organization
curl -H "$HEADER" "http://localhost:3000/api/org"

# Create organization
curl -X POST -H "$HEADER" -H "Content-Type: application/json" \
  "http://localhost:3000/api/orgs" \
  -d '{
    "name": "My Organization"
  }'

# Switch organization
curl -X POST -H "$HEADER" "http://localhost:3000/api/user/using/2"
```

### Alerting API

```bash
# Get alert rules
curl -H "$HEADER" "http://localhost:3000/api/ruler/grafana/api/v1/rules"

# Get specific rule group
curl -H "$HEADER" "http://localhost:3000/api/ruler/grafana/api/v1/rules/folder/group"

# Create/Update rule group
curl -X POST -H "$HEADER" -H "Content-Type: application/json" \
  "http://localhost:3000/api/ruler/grafana/api/v1/rules/folder" \
  -d @rule-group.json

# Delete rule group
curl -X DELETE -H "$HEADER" "http://localhost:3000/api/ruler/grafana/api/v1/rules/folder/group"

# Get contact points
curl -H "$HEADER" "http://localhost:3000/api/alertmanager/grafana/api/v1/config"

# Test contact point
curl -X POST -H "$HEADER" -H "Content-Type: application/json" \
  "http://localhost:3000/api/alertmanager/grafana/config/api/v1/receivers/test" \
  -d '{
    "name": "test-receiver",
    "grafana_managed_receiver_configs": [
      {
        "name": "test",
        "type": "slack",
        "settings": {
          "url": "https://hooks.slack.com/services/..."
        }
      }
    ]
  }'
```

## Plugins

### Installing Plugins

```bash
# Install plugin via CLI
grafana-cli plugins install grafana-clock-panel
grafana-cli plugins install grafana-piechart-panel
grafana-cli plugins install grafana-worldmap-panel
grafana-cli plugins install camptocamp-prometheus-alertmanager-datasource

# Install specific version
grafana-cli plugins install grafana-clock-panel 1.3.0

# Install from URL
grafana-cli --pluginUrl https://github.com/user/plugin/archive/main.zip plugins install plugin-name

# List installed plugins
grafana-cli plugins list-remote

# Update plugin
grafana-cli plugins update grafana-clock-panel

# Remove plugin
grafana-cli plugins remove grafana-clock-panel

# Install plugins with Docker
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -e GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-piechart-panel \
  grafana/grafana-oss
```

### Popular Plugins

```bash
# Panel plugins
grafana-clock-panel           # Clock panel
grafana-piechart-panel        # Pie chart panel
grafana-worldmap-panel        # World map panel
volkovlabs-echarts-panel      # Apache ECharts panel
volkovlabs-image-panel        # Dynamic image panel
petrslavotinek-carpetplot-panel  # Carpet plot panel

# Data source plugins
camptocamp-prometheus-alertmanager-datasource  # Alertmanager
grafana-azure-monitor-datasource               # Azure Monitor
grafana-googlesheets-datasource                # Google Sheets
grafana-github-datasource                      # GitHub
grafana-json-datasource                        # JSON API

# App plugins
grafana-kubernetes-app        # Kubernetes monitoring
grafana-synthetic-monitoring   # Synthetic monitoring
volkovlabs-rss-datasource     # RSS/Atom feeds
```

### Plugin Configuration

```ini
# grafana.ini
[plugins]
# Enable plugins
enable_alpha = true
allow_loading_unsigned_plugins = my-custom-plugin

# Plugin marketplace
marketplace_url = https://grafana.com/grafana/plugins/

# Plugin catalog
plugin_catalog_url = https://grafana.com/api/plugins

# Plugin update check
check_for_plugin_updates = true
```

## Performance Optimization

### Query Optimization

```json
{
  "targets": [
    {
      "expr": "rate(http_requests_total[5m])",
      "interval": "30s",
      "maxDataPoints": 1000,
      "step": 30
    }
  ],
  "options": {
    "series": {
      "limit": 100
    }
  }
}
```

### Caching Configuration

```ini
[caching]
# Enable query result caching
enabled = true

# Cache TTL
ttl = 5m

# Maximum cache size
max_value_mb = 25
```

### Database Optimization

```ini
[database]
# Use external database for better performance
type = mysql
host = mysql:3306
name = grafana
user = grafana
password = password

# Connection pooling
max_idle_conn = 2
max_open_conn = 100
conn_max_lifetime = 14400

# Query logging
log_queries = false
```

### Resource Limits

```yaml
# kubernetes deployment
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

## Monitoring Grafana

### Grafana Metrics

```promql
# Query performance
grafana_api_dataproxy_request_duration_seconds
grafana_api_request_duration_seconds
grafana_database_conn_idle
grafana_database_conn_inuse
grafana_database_conn_max_open

# Dashboard metrics
grafana_stat_totals_dashboard
grafana_stat_totals_user
grafana_stat_totals_org
grafana_stat_totals_datasource

# Alert metrics
grafana_alerting_rule_evaluations_total
grafana_alerting_rule_evaluation_duration_seconds
grafana_alerting_notifications_sent_total

# Memory and CPU
process_resident_memory_bytes{job="grafana"}
process_cpu_seconds_total{job="grafana"}
```

### Health Check

```bash
# Health endpoint
curl http://localhost:3000/api/health

# Metrics endpoint
curl http://localhost:3000/metrics

# Ready endpoint
curl http://localhost:3000/api/ready
```

## Security

### Authentication Configuration

```ini
[auth]
# Disable login form
disable_login_form = false

# Disable signups
disable_signout_menu = false

# OAuth configuration
[auth.github]
enabled = true
allow_sign_up = true
client_id = your_github_client_id
client_secret = your_github_client_secret
scopes = user:email,read:org
auth_url = https://github.com/login/oauth/authorize
token_url = https://github.com/login/oauth/access_token
api_url = https://api.github.com/user
team_ids =
allowed_organizations =

# LDAP configuration
[auth.ldap]
enabled = true
config_file = /etc/grafana/ldap.toml
allow_sign_up = true
```

### LDAP Configuration

```toml
# /etc/grafana/ldap.toml
[[servers]]
host = "ldap.example.com"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = false

bind_dn = "cn=admin,dc=grafana,dc=org"
bind_password = 'grafana'

search_filter = "(cn=%s)"
search_base_dns = ["dc=grafana,dc=org"]

[servers.attributes]
name = "givenName"
surname = "sn"
username = "cn"
member_of = "memberOf"
email = "email"

[[servers.group_mappings]]
group_dn = "cn=admins,dc=grafana,dc=org"
org_role = "Admin"
grafana_admin = true

[[servers.group_mappings]]
group_dn = "cn=users,dc=grafana,dc=org"
org_role = "Editor"

[[servers.group_mappings]]
group_dn = "*"
org_role = "Viewer"
```

### TLS Configuration

```ini
[server]
protocol = https
cert_file = /path/to/cert.pem
cert_key = /path/to/cert.key

# TLS settings
tls_min_version = "1.2"
tls_cipher_suites = [
  "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
  "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
  "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
  "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"
]
```

## Troubleshooting

### Common Issues

```bash
# Check Grafana logs
docker logs grafana
sudo journalctl -u grafana-server -f
tail -f /var/log/grafana/grafana.log

# Check configuration
grafana-server -config /etc/grafana/grafana.ini -check-config

# Test data source connection
curl -X POST -H "Authorization: Bearer $API_KEY" \
  "http://localhost:3000/api/datasources/1/health"

# Check plugin status
grafana-cli plugins list-remote

# Database connection issues
telnet mysql-host 3306
nc -zv postgres-host 5432

# Permission issues
ls -la /var/lib/grafana
chown -R grafana:grafana /var/lib/grafana

# Memory issues
free -h
top -p $(pgrep grafana)
```

### Debug Mode

```ini
[log]
mode = console file
level = debug

[log.console]
level = debug
format = console

[log.file]
level = debug
format = text
log_rotate = true
max_lines = 1000000
max_size_shift = 28
daily_rotate = true
max_days = 7
```

### Performance Issues

```bash
# Check query performance
curl -H "Authorization: Bearer $API_KEY" \
  "http://localhost:3000/api/datasources/proxy/1/api/v1/query?query=up" \
  -w "Time: %{time_total}s\n"

# Monitor Grafana metrics
curl http://localhost:3000/metrics | grep grafana_

# Check database performance
SELECT * FROM dashboard WHERE created > NOW() - INTERVAL 1 DAY;
SHOW PROCESSLIST;

# Memory profiling
go tool pprof http://localhost:3000/debug/pprof/heap
go tool pprof http://localhost:3000/debug/pprof/profile
```

## Best Practices

### Dashboard Design

```json
{
  "dashboard": {
    "title": "Service Overview",
    "description": "High-level metrics for service monitoring",
    "tags": ["service", "overview", "production"],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "30s",
    "panels": [
      {
        "title": "SLI Metrics",
        "type": "row",
        "collapsed": false
      },
      {
        "title": "Error Rate",
        "type": "stat",
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "steps": [
                {"color": "green", "value": null},
                {"color": "yellow", "value": 1},
                {"color": "red", "value": 5}
              ]
            }
          }
        }
      }
    ]
  }
}
```

### Variable Usage

```promql
# Use variables in queries
sum(rate(http_requests_total{instance=~"$instance", job="$job"}[$interval]))

# Multi-value variables
sum(rate(http_requests_total{instance=~"$instance"}[5m])) by (instance)

# Regex variables
label_values(up{job=~"$service.*"}, instance)
```

### Organization Structure

```yaml
# Organization hierarchy
Organizations:
  - name: "Production"
    users:
      - admin: ["admin@company.com"]
      - editor: ["dev@company.com"]
      - viewer: ["stakeholder@company.com"]
    
  - name: "Development"
    users:
      - admin: ["dev@company.com"]
      - editor: ["intern@company.com"]

# Folder structure
Folders:
  - "Infrastructure"
  - "Applications"
  - "Business Metrics"
  - "SLI/SLO Dashboards"
```

### Naming Conventions

```text
# Dashboard naming
[Environment] Service - Overview
[Production] API Gateway - Overview
[Staging] Database - Performance

# Panel naming
Availability (99.9%)
Latency P95 (ms)
Error Rate (%)
Throughput (req/s)

# Variable naming
$environment
$service
$instance
$interval
$timerange
```

---

*For more detailed information, visit the [official Grafana documentation](https://grafana.com/docs/)*
