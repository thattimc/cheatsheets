# OpenCost Cheatsheet

OpenCost is a vendor-neutral open source project for measuring and allocating
infrastructure and container costs in Kubernetes environments. It provides real-time cost
monitoring with Kubernetes native concepts.

## Overview

### Key Features

- **Real-time Cost Monitoring** - Track costs as workloads run
- **Kubernetes Native** - Uses labels, namespaces, and other K8s concepts
- **Multi-Cloud Support** - Works with AWS, GCP, Azure, on-premises
- **Cost Allocation** - Allocate shared costs to teams and applications
- **Budget Alerts** - Set up cost-based alerts and notifications
- **API-First** - RESTful API for cost data integration
- **Open Source** - No vendor lock-in, community-driven
- **Prometheus Integration** - Built on Prometheus metrics

### Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Kubernetes    │    │    OpenCost     │    │   Monitoring    │
│                 │    │                 │    │                 │
│ • Pods          │───►│ • Cost Model    │───►│ • Prometheus    │
│ • Nodes         │    │ • API Server    │    │ • Grafana       │
│ • PVs           │    │ • UI            │    │ • Alertmanager  │
│ • Services      │    │ • Allocation    │    │ • Custom Apps   │
│ • Namespaces    │    │ • Reports       │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                       ┌─────────────────┐
                       │ Cloud Providers │
                       │                 │
                       │ • AWS Pricing   │
                       │ • GCP Pricing   │
                       │ • Azure Pricing │
                       │ • Custom Rates  │
                       └─────────────────┘
```

### Core Concepts

- **Cost Model** - Calculates infrastructure costs based on resource usage
- **Allocation** - Distributes costs to pods, namespaces, labels, etc.
- **Assets** - Cloud resources like nodes, disks, load balancers
- **Efficiency** - Measures resource utilization and waste
- **Budgets** - Cost limits and alerting thresholds
- **Windows** - Time periods for cost analysis

## Installation

### Helm Installation (Recommended)

```bash
# Add OpenCost Helm repository
helm repo add opencost https://opencost.github.io/opencost-helm-chart
helm repo update

# Install OpenCost
helm install opencost opencost/opencost --namespace opencost --create-namespace

# Install with custom values
helm install opencost opencost/opencost \
  --namespace opencost \
  --create-namespace \
  --set prometheus.external.url=http://prometheus:9090 \
  --set ui.enabled=true

# Upgrade OpenCost
helm upgrade opencost opencost/opencost --namespace opencost

# Get status
helm status opencost --namespace opencost
```

### Kubernetes Manifests

```bash
# Install OpenCost using kubectl
kubectl apply --namespace opencost -f https://raw.githubusercontent.com/opencost/opencost/develop/kubernetes/opencost.yaml

# Create namespace first
kubectl create namespace opencost

# Apply manifests
kubectl apply -f https://raw.githubusercontent.com/opencost/opencost/develop/kubernetes/opencost.yaml -n opencost

# Check installation
kubectl get pods -n opencost
kubectl get svc -n opencost
```

### Custom Installation

```yaml
# opencost-values.yaml
opencost:
  exporter:
    # Cloud provider configuration
    cloudProviderApiKey: "your-api-key"
    
    # Prometheus configuration
    prometheus:
      external:
        url: "http://prometheus-server.monitoring.svc.cluster.local:80"
    
    # Default cluster costs (if no cloud pricing)
    defaultClusterId: "default-cluster"
    
    # Resource costs (per hour)
    cpuCost: 0.031611
    ramCost: 0.004237
    gpuCost: 0.95
    
    # Storage costs (per GB per hour)
    storageClassMapping:
      default: 0.00005
      fast-ssd: 0.0001
      slow: 0.00002

ui:
  enabled: true
  image:
    repository: quay.io/kubecost1/opencost-ui
    tag: prod-1.108.1

prometheus:
  external:
    enabled: true
    url: "http://prometheus-server.monitoring.svc.cluster.local:80"

service:
  type: ClusterIP
  port: 9003
  targetPort: 9003

serviceAccount:
  create: true
  name: opencost
```

```bash
# Install with custom values
helm install opencost opencost/opencost \
  --namespace opencost \
  --create-namespace \
  --values opencost-values.yaml
```

### Access OpenCost UI

```bash
# Port forward to access UI
kubectl port-forward --namespace opencost svc/opencost 9090:9090

# Access UI at http://localhost:9090

# Using LoadBalancer (cloud environments)
kubectl patch svc opencost -n opencost -p '{"spec": {"type": "LoadBalancer"}}'

# Get external IP
kubectl get svc opencost -n opencost

# Using Ingress
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: opencost-ingress
  namespace: opencost
spec:
  rules:
  - host: opencost.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: opencost
            port:
              number: 9090
EOF
```

## Configuration

### Environment Variables

```yaml
# OpenCost configuration via environment variables
env:
  # Cloud Provider Configuration
  - name: CLOUD_PROVIDER_API_KEY
    value: "your-api-key"
  
  - name: CLOUD_PROVIDER
    value: "aws"  # or "gcp", "azure", "custom"
  
  # Prometheus Configuration
  - name: PROMETHEUS_SERVER_ENDPOINT
    value: "http://prometheus-server:80"
  
  # Cluster Configuration
  - name: CLUSTER_ID
    value: "production-cluster"
  
  # Custom Pricing
  - name: CPU_COST_PER_CORE_HOUR
    value: "0.031611"
  
  - name: RAM_COST_PER_GB_HOUR
    value: "0.004237"
  
  - name: GPU_COST_PER_HOUR
    value: "0.95"
  
  # Storage Costs
  - name: STORAGE_COST_PER_GB_HOUR
    value: "0.00005"
  
  # Currency
  - name: CURRENCY
    value: "USD"
  
  # Update frequency
  - name: EMIT_KSM_V1_METRICS
    value: "false"
  
  - name: EMIT_KSM_V1_METRICS_ONLY
    value: "false"
  
  # Logging
  - name: LOG_LEVEL
    value: "info"
```

### Cloud Provider Setup

#### AWS Configuration

```yaml
env:
  - name: CLOUD_PROVIDER
    value: "aws"
  
  - name: AWS_ACCESS_KEY_ID
    valueFrom:
      secretKeyRef:
        name: aws-credentials
        key: access-key-id
  
  - name: AWS_SECRET_ACCESS_KEY
    valueFrom:
      secretKeyRef:
        name: aws-credentials
        key: secret-access-key
  
  - name: AWS_REGION
    value: "us-west-2"
  
  # For AWS Cost and Usage Report
  - name: AWS_CUR_S3_BUCKET
    value: "my-cur-bucket"
  
  - name: AWS_CUR_S3_PATH
    value: "cur-data/"
```

```bash
# Create AWS credentials secret
kubectl create secret generic aws-credentials \
  --from-literal=access-key-id=YOUR_ACCESS_KEY \
  --from-literal=secret-access-key=YOUR_SECRET_KEY \
  -n opencost
```

#### GCP Configuration

```yaml
env:
  - name: CLOUD_PROVIDER
    value: "gcp"
  
  - name: GOOGLE_APPLICATION_CREDENTIALS
    value: "/var/secrets/google/key.json"
  
  - name: GCP_PROJECT_ID
    value: "my-project-id"

volumeMounts:
  - name: gcp-key
    mountPath: /var/secrets/google
    readOnly: true

volumes:
  - name: gcp-key
    secret:
      secretName: gcp-service-account
```

```bash
# Create GCP service account secret
kubectl create secret generic gcp-service-account \
  --from-file=key.json=./service-account-key.json \
  -n opencost
```

#### Azure Configuration

```yaml
env:
  - name: CLOUD_PROVIDER
    value: "azure"
  
  - name: AZURE_SUBSCRIPTION_ID
    value: "your-subscription-id"
  
  - name: AZURE_CLIENT_ID
    valueFrom:
      secretKeyRef:
        name: azure-credentials
        key: client-id
  
  - name: AZURE_CLIENT_SECRET
    valueFrom:
      secretKeyRef:
        name: azure-credentials
        key: client-secret
  
  - name: AZURE_TENANT_ID
    value: "your-tenant-id"
```

### Custom Pricing Configuration

```yaml
# For on-premises or custom pricing
env:
  - name: CLOUD_PROVIDER
    value: "custom"
  
  # Node pricing
  - name: CPU_COST_PER_CORE_HOUR
    value: "0.031611"  # $0.031611 per core per hour
  
  - name: RAM_COST_PER_GB_HOUR
    value: "0.004237"  # $0.004237 per GB per hour
  
  - name: GPU_COST_PER_HOUR
    value: "0.95"      # $0.95 per GPU per hour
  
  # Storage pricing
  - name: STORAGE_COST_PER_GB_HOUR
    value: "0.00005"   # $0.00005 per GB per hour
  
  # Network pricing
  - name: NETWORK_COST_PER_GB
    value: "0.01"      # $0.01 per GB
  
  # Load balancer pricing
  - name: LOAD_BALANCER_COST_PER_HOUR
    value: "0.025"     # $0.025 per hour
```

## API Usage

### Basic API Endpoints

```bash
# Base URL
OPENCOST_URL="http://localhost:9090"

# Health check
curl "$OPENCOST_URL/healthz"

# Model info
curl "$OPENCOST_URL/model"

# Allocation API - Get costs by namespace
curl "$OPENCOST_URL/allocation?window=1d&aggregate=namespace"

# Allocation API - Get costs by pod
curl "$OPENCOST_URL/allocation?window=7d&aggregate=pod"

# Assets API - Get node costs
curl "$OPENCOST_URL/assets?window=1d&aggregate=type"

# Cloud costs
curl "$OPENCOST_URL/cloudCost?window=7d&aggregate=service"
```

### Allocation API Examples

```bash
# Get allocation data for last 24 hours
curl "$OPENCOST_URL/allocation?window=1d"

# Get allocation by namespace for last week
curl "$OPENCOST_URL/allocation?window=7d&aggregate=namespace"

# Get allocation by label
curl "$OPENCOST_URL/allocation?window=1d&aggregate=label:app"

# Get allocation by multiple dimensions
curl "$OPENCOST_URL/allocation?window=1d&aggregate=namespace,label:app"

# Filter by namespace
curl "$OPENCOST_URL/allocation?window=1d&filter=namespace:default"

# Filter by label
curl "$OPENCOST_URL/allocation?window=1d&filter=label[app]:nginx"

# Accumulate costs (no time breakdown)
curl "$OPENCOST_URL/allocation?window=7d&aggregate=namespace&accumulate=true"

# Step parameter for resolution
curl "$OPENCOST_URL/allocation?window=7d&step=1d&aggregate=namespace"

# Share idle costs
curl "$OPENCOST_URL/allocation?window=1d&shareIdle=true"

# Include external costs
curl "$OPENCOST_URL/allocation?window=1d&includeExternal=true"
```

### Assets API Examples

```bash
# Get all assets for last 24 hours
curl "$OPENCOST_URL/assets?window=1d"

# Get assets by type
curl "$OPENCOST_URL/assets?window=1d&aggregate=type"

# Get node assets
curl "$OPENCOST_URL/assets?window=1d&filter=type:Node"

# Get storage assets
curl "$OPENCOST_URL/assets?window=1d&filter=type:Disk"

# Get load balancer assets
curl "$OPENCOST_URL/assets?window=1d&filter=type:LoadBalancer"

# Accumulate asset costs
curl "$OPENCOST_URL/assets?window=7d&accumulate=true"
```

### Cloud Cost API Examples

```bash
# Get cloud costs by service
curl "$OPENCOST_URL/cloudCost?window=7d&aggregate=service"

# Get cloud costs by account
curl "$OPENCOST_URL/cloudCost?window=30d&aggregate=account"

# Filter by service
curl "$OPENCOST_URL/cloudCost?window=7d&filter=service:AmazonEC2"

# Get costs with labels
curl "$OPENCOST_URL/cloudCost?window=7d&aggregate=label:Environment"
```

### Advanced API Usage

```bash
# Complex allocation query
curl -G "$OPENCOST_URL/allocation" \
  --data-urlencode "window=7d" \
  --data-urlencode "aggregate=namespace,label:app,label:env" \
  --data-urlencode "filter=namespace:production|staging" \
  --data-urlencode "step=1d" \
  --data-urlencode "shareIdle=true"

# Get top 10 most expensive namespaces
curl "$OPENCOST_URL/allocation?window=30d&aggregate=namespace&accumulate=true" | \
  jq '.data[0] | to_entries | sort_by(-.value.totalCost) | .[0:10]'

# Calculate efficiency (cost vs usage)
curl "$OPENCOST_URL/allocation?window=1d&includeEfficiency=true"

# Get costs with shared resources allocated
curl "$OPENCOST_URL/allocation?window=1d&shareNamespaces=kube-system,monitoring"
```

## Monitoring and Alerting

### Prometheus Metrics

```promql
# Key OpenCost metrics

# Total cluster costs
sum(opencost_cluster_management_cost_total)

# Namespace costs
sum by (namespace) (opencost_allocation_cpu_cost)
sum by (namespace) (opencost_allocation_ram_cost)
sum by (namespace) (opencost_allocation_storage_cost)

# Pod costs
sum by (namespace, pod) (
  opencost_allocation_cpu_cost + 
  opencost_allocation_ram_cost + 
  opencost_allocation_storage_cost
)

# Node costs
sum by (node) (opencost_node_cost_hourly)

# Efficiency metrics
opencost_allocation_cpu_efficiency
opencost_allocation_ram_efficiency

# Resource requests vs usage
opencost_allocation_cpu_cost / opencost_allocation_cpu_hours
opencost_allocation_ram_cost / opencost_allocation_ram_bytes
```

### Grafana Dashboard Queries

```promql
# Total daily costs
sum(increase(opencost_allocation_total_cost[24h]))

# Top 10 most expensive namespaces
topk(10, sum by (namespace) (
  increase(opencost_allocation_total_cost[24h])
))

# Cost trend over time
sum(rate(opencost_allocation_total_cost[5m])) * 86400  # Daily rate

# CPU efficiency by namespace
avg by (namespace) (opencost_allocation_cpu_efficiency)

# Memory efficiency by namespace
avg by (namespace) (opencost_allocation_ram_efficiency)

# Storage costs by storage class
sum by (storageclass) (opencost_allocation_storage_cost)

# Idle resource costs
sum(opencost_cluster_idle_cost)

# Cost per request (if you have request metrics)
sum(rate(opencost_allocation_total_cost[5m])) / 
sum(rate(http_requests_total[5m]))
```

### Alerting Rules

```yaml
# opencost-alerts.yaml
groups:
  - name: opencost.rules
    rules:
      - alert: HighNamespaceCost
        expr: |
          (
            sum by (namespace) (
              increase(opencost_allocation_total_cost[24h])
            ) > 100
          )
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High cost detected for namespace {{ $labels.namespace }}"
          description: "Namespace {{ $labels.namespace }} has cost ${{ $value }} in the last 24 hours"
      
      - alert: LowResourceEfficiency
        expr: |
          (
            avg by (namespace) (opencost_allocation_cpu_efficiency) < 0.3
          )
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "Low CPU efficiency in namespace {{ $labels.namespace }}"
          description: "CPU efficiency is {{ $value | humanizePercentage }} in namespace {{ $labels.namespace }}"
      
      - alert: UnexpectedCostIncrease
        expr: |
          (
            (
              sum(increase(opencost_allocation_total_cost[24h]))
              -
              sum(increase(opencost_allocation_total_cost[24h] offset 24h))
            )
            /
            sum(increase(opencost_allocation_total_cost[24h] offset 24h))
          ) > 0.5
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Unexpected cost increase detected"
          description: "Daily costs increased by {{ $value | humanizePercentage }} compared to yesterday"
      
      - alert: HighIdleCosts
        expr: |
          (
            sum(opencost_cluster_idle_cost) / 
            sum(opencost_cluster_management_cost_total)
          ) > 0.3
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High idle resource costs"
          description: "Idle costs are {{ $value | humanizePercentage }} of total cluster costs"
```

## Cost Optimization

### Resource Right-sizing

```bash
# Get resource efficiency data
curl "$OPENCOST_URL/allocation?window=7d&includeEfficiency=true" | \
  jq '.data[] | to_entries[] | select(.value.cpuEfficiency < 0.5) | {namespace: .key, cpuEfficiency: .value.cpuEfficiency, cost: .value.totalCost}'

# Find overprovisioned workloads
curl "$OPENCOST_URL/allocation?window=7d&aggregate=pod&includeEfficiency=true" | \
  jq '.data[] | to_entries[] | select(.value.ramEfficiency < 0.3) | {pod: .key, ramEfficiency: .value.ramEfficiency, ramCost: .value.ramCost}'

# Calculate potential savings from right-sizing
curl "$OPENCOST_URL/allocation?window=30d&aggregate=namespace&includeEfficiency=true" | \
  jq '.data[] | to_entries[] | {namespace: .key, potentialSavings: (.value.cpuCost * (1 - .value.cpuEfficiency) + .value.ramCost * (1 - .value.ramEfficiency))}'
```

### Spot Instance Recommendations

```promql
# Identify workloads suitable for spot instances
# (stateless, fault-tolerant workloads)
sum by (namespace, deployment) (
  opencost_allocation_total_cost
) and on (namespace) (
  label_replace(
    kube_deployment_spec_replicas > 1,
    "deployment", "$1", "deployment", "(.*)"
  )
)
```

### Storage Optimization

```bash
# Find unused persistent volumes
curl "$OPENCOST_URL/assets?window=7d&filter=type:Disk" | \
  jq '.data[] | to_entries[] | select(.value.adjustment == 0) | {volume: .key, cost: .value.totalCost}'

# Identify expensive storage classes
curl "$OPENCOST_URL/allocation?window=30d&aggregate=storageclass&accumulate=true" | \
  jq '.data[0] | to_entries | sort_by(-.value.storageCost) | .[0:5]'
```

## Custom Metrics and Dashboards

### Custom Metrics Configuration

```yaml
# Add custom metrics to OpenCost
apiVersion: v1
kind: ConfigMap
metadata:
  name: opencost-custom-metrics
  namespace: opencost
data:
  custom-metrics.yaml: |
    customMetrics:
      - name: "requests_per_dollar"
        query: "sum(rate(http_requests_total[5m])) / sum(rate(opencost_allocation_total_cost[5m]))"
        unit: "requests/$"
      
      - name: "cost_per_user"
        query: "sum(rate(opencost_allocation_total_cost[5m])) / sum(active_users)"
        unit: "$/user"
      
      - name: "efficiency_score"
        query: "(avg(opencost_allocation_cpu_efficiency) + avg(opencost_allocation_ram_efficiency)) / 2"
        unit: "ratio"
```

### Grafana Dashboard JSON

```json
{
  "dashboard": {
    "id": null,
    "title": "OpenCost Overview",
    "description": "Kubernetes cost monitoring with OpenCost",
    "tags": ["opencost", "kubernetes", "cost"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Total Daily Cost",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(increase(opencost_allocation_total_cost[24h]))",
            "legendFormat": "Total Cost"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "currencyUSD",
            "decimals": 2
          }
        }
      },
      {
        "id": 2,
        "title": "Cost by Namespace",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum by (namespace) (rate(opencost_allocation_total_cost[5m])) * 86400",
            "legendFormat": "{{ namespace }}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "currencyUSD"
          }
        }
      },
      {
        "id": 3,
        "title": "Resource Efficiency",
        "type": "timeseries",
        "targets": [
          {
            "expr": "avg by (namespace) (opencost_allocation_cpu_efficiency)",
            "legendFormat": "CPU Efficiency - {{ namespace }}"
          },
          {
            "expr": "avg by (namespace) (opencost_allocation_ram_efficiency)",
            "legendFormat": "RAM Efficiency - {{ namespace }}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percentunit",
            "min": 0,
            "max": 1
          }
        }
      },
      {
        "id": 4,
        "title": "Top Cost Contributors",
        "type": "table",
        "targets": [
          {
            "expr": "topk(10, sum by (namespace) (increase(opencost_allocation_total_cost[24h])))",
            "format": "table",
            "instant": true
          }
        ]
      }
    ],
    "time": {
      "from": "now-24h",
      "to": "now"
    },
    "refresh": "5m"
  }
}
```

## Budget Management

### Budget Configuration

```yaml
# Budget alerts via Prometheus rules
apiVersion: v1
kind: ConfigMap
metadata:
  name: opencost-budgets
  namespace: opencost
data:
  budgets.yaml: |
    budgets:
      - name: "development-monthly"
        filters:
          - namespace: "development"
        budget: 1000  # $1000 per month
        alertThresholds:
          - 50   # 50% of budget
          - 80   # 80% of budget
          - 100  # 100% of budget
      
      - name: "production-monthly"
        filters:
          - namespace: "production"
        budget: 5000  # $5000 per month
        alertThresholds:
          - 75
          - 90
          - 100
      
      - name: "team-alpha-quarterly"
        filters:
          - label: "team=alpha"
        budget: 10000  # $10000 per quarter
        period: "quarterly"
        alertThresholds:
          - 60
          - 85
          - 100
```

### Budget Monitoring Queries

```promql
# Monthly budget utilization
(
  sum(increase(opencost_allocation_total_cost{namespace="production"}[30d]))
  / 5000  # Budget amount
) * 100

# Projected monthly spend
sum(rate(opencost_allocation_total_cost{namespace="development"}[7d])) * 86400 * 30

# Budget burn rate
sum(rate(opencost_allocation_total_cost[1d])) * 86400

# Days until budget exhausted
(
  5000 - sum(increase(opencost_allocation_total_cost{namespace="production"}[30d]))
) / (
  sum(rate(opencost_allocation_total_cost{namespace="production"}[7d])) * 86400
)
```

## Multi-tenancy and RBAC

### RBAC Configuration

```yaml
# Team-based RBAC
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: opencost-reader
rules:
- apiGroups: [""]
  resources: ["pods", "nodes", "namespaces", "persistentvolumes", "persistentvolumeclaims"]
  verbs: ["get", "list"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["*"]
  verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: opencost-team-alpha
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: opencost-reader
subjects:
- kind: User
  name: team-alpha-user
  apiGroup: rbac.authorization.k8s.io
```

### Namespace-scoped Cost Access

```bash
# Create service account for team
kubectl create serviceaccount team-alpha-opencost -n team-alpha

# Bind to role with namespace access only
kubectl create rolebinding team-alpha-opencost-binding \
  --clusterrole=opencost-reader \
  --serviceaccount=team-alpha:team-alpha-opencost \
  -n team-alpha

# Get team-specific costs
TOKEN=$(kubectl get secret $(kubectl get sa team-alpha-opencost -n team-alpha -o jsonpath='{.secrets[0].name}') -n team-alpha -o jsonpath='{.data.token}' | base64 -d)

curl -H "Authorization: Bearer $TOKEN" \
  "$OPENCOST_URL/allocation?window=7d&filter=namespace:team-alpha"
```

## Troubleshooting

### Common Issues

```bash
# Check OpenCost pods
kubectl get pods -n opencost
kubectl describe pod -l app=opencost -n opencost

# Check logs
kubectl logs -l app=opencost -n opencost --tail=100
kubectl logs -l app=opencost -n opencost -c cost-model

# Check Prometheus connectivity
kubectl exec -it deployment/opencost -n opencost -- \
  curl http://prometheus-server.monitoring.svc.cluster.local:80/api/v1/query?query=up

# Verify metrics are being scraped
curl "$OPENCOST_URL/metrics" | grep opencost

# Check if cost data is available
curl "$OPENCOST_URL/allocation?window=1h" | jq '.code'

# Validate configuration
kubectl exec -it deployment/opencost -n opencost -- \
  cat /etc/opencost/config.yaml
```

### Debugging Cost Calculations

```bash
# Check node costs
curl "$OPENCOST_URL/assets?window=1d&filter=type:Node" | \
  jq '.data[] | to_entries[] | {node: .key, hourlyCost: .value.totalCost}'

# Verify resource usage metrics
curl "$OPENCOST_URL/model/prometheusQuery" \
  -d 'query=container_cpu_usage_seconds_total'

# Check pricing data
curl "$OPENCOST_URL/model/pricing" | jq '.data'

# Validate allocation calculations
curl "$OPENCOST_URL/allocation?window=1h&resolution=1m" | \
  jq '.data[] | to_entries[0] | {time: .key, allocations: .value | length}'
```

### Performance Issues

```bash
# Check memory usage
kubectl top pod -n opencost

# Monitor API response times
time curl "$OPENCOST_URL/allocation?window=30d&aggregate=namespace"

# Check Prometheus query performance
curl "$OPENCOST_URL/model/prometheusQuery" \
  -d 'query=sum(container_cpu_usage_seconds_total)' \
  -w "Time: %{time_total}s\n"

# Verify storage usage
kubectl exec -it deployment/opencost -n opencost -- df -h
```

## Best Practices

### Label Strategy

```yaml
# Good labeling practices for cost allocation
metadata:
  labels:
    # Team ownership
    team: "platform"
    
    # Cost center
    cost-center: "engineering"
    
    # Environment
    environment: "production"
    
    # Application
    app: "web-server"
    
    # Version for tracking costs across deployments
    version: "v1.2.3"
    
    # Business unit
    business-unit: "retail"
    
    # Project for chargeback
    project: "mobile-app"
```

### Resource Configuration

```yaml
# Always set resource requests and limits
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

# Use appropriate node selectors for cost optimization
nodeSelector:
  node-type: "spot"  # For fault-tolerant workloads

# Use taints and tolerations for dedicated nodes
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "gpu-workload"
  effect: "NoSchedule"
```

### Monitoring Setup

```yaml
# Monitor key cost metrics
prometheus:
  additionalScrapeConfigs: |
    - job_name: 'opencost'
      static_configs:
        - targets: ['opencost.opencost.svc.cluster.local:9003']
      metrics_path: /metrics
      scrape_interval: 30s
      scrape_timeout: 10s

# Set up cost-based alerts
alerting:
  rules:
    - alert: DailyCostThreshold
      expr: sum(increase(opencost_allocation_total_cost[24h])) > 1000
      for: 1h
      labels:
        severity: warning
      annotations:
        summary: "Daily cost threshold exceeded"
```

### Data Retention

```yaml
# Configure appropriate data retention
opencost:
  config:
    # Keep detailed data for 7 days
    retentionResolution: "1m"
    retentionPeriod: "7d"
    
    # Keep hourly data for 30 days
    downsampleResolution: "1h"
    downsamplePeriod: "30d"
    
    # Keep daily data for 1 year
    archiveResolution: "1d"
    archivePeriod: "365d"
```

---

*For more detailed information, visit the [official OpenCost documentation](https://www.opencost.io/docs/)*
