# RKE2 Cheatsheet

RKE2 (Rancher Kubernetes Engine 2) is a CNCF-certified Kubernetes distribution that focuses on security and compliance. It's the successor to RKE and is designed for production environments.

## Overview

### Key Features
- **FIPS 140-2 Compliance** - Federal Information Processing Standard compliance
- **CIS Hardening** - Center for Internet Security benchmark compliance
- **SELinux Support** - Security-Enhanced Linux integration
- **etcd Snapshots** - Automated backup and restore
- **Embedded Components** - containerd, runc, CNI plugins
- **Air-gapped Support** - Offline installation capabilities

### Architecture
- **Server Nodes** - Control plane components (etcd, API server, scheduler, controller)
- **Agent Nodes** - Worker nodes running kubelet and kube-proxy
- **Embedded etcd** - Highly available datastore
- **Load Balancer** - Optional external load balancer for HA

## Installation

### Prerequisites
```bash
# System requirements
# - Linux kernel 3.10+
# - 2GB+ RAM for server nodes
# - 1GB+ RAM for agent nodes
# - 10GB+ disk space

# Disable firewall or configure ports
sudo systemctl disable firewalld
sudo systemctl stop firewalld

# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Set SELinux to permissive (if using SELinux)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### Install RKE2 Server (First Node)
```bash
# Download and install RKE2
curl -sfL https://get.rke2.io | sh -

# Enable and start RKE2 server
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service

# Check status
sudo systemctl status rke2-server.service

# View logs
sudo journalctl -u rke2-server -f
```

### Install RKE2 Agent (Worker Nodes)
```bash
# Download and install RKE2
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -

# Configure agent to join cluster
sudo mkdir -p /etc/rancher/rke2/
sudo tee /etc/rancher/rke2/config.yaml > /dev/null <<EOF
server: https://<server-ip>:9345
token: <node-token>
EOF

# Enable and start RKE2 agent
sudo systemctl enable rke2-agent.service
sudo systemctl start rke2-agent.service

# Check status
sudo systemctl status rke2-agent.service
```

### Get Node Token
```bash
# On server node, get the token
sudo cat /var/lib/rancher/rke2/server/node-token

# Alternative: use kubectl
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
kubectl get secret -n kube-system rke2-serving -o jsonpath='{.data.token}' | base64 -d
```

## Configuration

### Server Configuration (/etc/rancher/rke2/config.yaml)
```yaml
# Basic server configuration
bind-address: 0.0.0.0
cluster-cidr: 10.42.0.0/16
service-cidr: 10.43.0.0/16
cluster-dns: 10.43.0.10
cluster-domain: cluster.local

# TLS configuration
tls-san:
  - "rke2-server.example.com"
  - "192.168.1.100"
  - "10.0.0.100"

# etcd configuration
etcd-snapshot-schedule-cron: "0 */12 * * *"
etcd-snapshot-retention: 5
etcd-snapshot-dir: /var/lib/rancher/rke2/server/db/snapshots

# Networking
cni: canal
cluster-cidr: 10.42.0.0/16
service-cidr: 10.43.0.0/16
disable-network-policy: false

# Security
protect-kernel-defaults: true
secrets-encryption: true

# Disable components
disable:
  - traefik
  - servicelb

# Node labels and taints
node-label:
  - "node-type=server"
node-taint:
  - "CriticalAddonsOnly=true:NoExecute"

# Kubelet configuration
kubelet-arg:
  - "max-pods=250"
  - "eviction-hard=memory.available<500Mi"

# Kube-apiserver configuration
kube-apiserver-arg:
  - "audit-log-maxage=30"
  - "audit-log-maxbackup=10"
  - "audit-log-maxsize=100"

# Data directory
data-dir: /var/lib/rancher/rke2
```

### Agent Configuration (/etc/rancher/rke2/config.yaml)
```yaml
# Server connection
server: https://rke2-server.example.com:9345
token: <node-token>

# Node configuration
node-label:
  - "node-type=worker"
  - "workload=general"
node-taint:
  - "worker=true:NoSchedule"

# Kubelet configuration
kubelet-arg:
  - "max-pods=110"
  - "eviction-hard=memory.available<200Mi"

# Data directory
data-dir: /var/lib/rancher/rke2

# Private registry
system-default-registry: "registry.example.com"
```

### High Availability Setup
```yaml
# First server node
cluster-init: true
tls-san:
  - "rke2-cluster.example.com"
  - "192.168.1.100"
  - "192.168.1.101"
  - "192.168.1.102"

# Additional server nodes
server: https://192.168.1.100:9345
token: <node-token>
tls-san:
  - "rke2-cluster.example.com"
  - "192.168.1.100"
  - "192.168.1.101"
  - "192.168.1.102"
```

## Cluster Management

### kubectl Configuration
```bash
# Set up kubectl access
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml

# Copy kubeconfig to user directory
mkdir -p ~/.kube
sudo cp /etc/rancher/rke2/rke2.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
chmod 600 ~/.kube/config

# Verify cluster access
kubectl get nodes
kubectl get pods -A
```

### Cluster Information
```bash
# Get cluster info
kubectl cluster-info

# Get node details
kubectl get nodes -o wide
kubectl describe node <node-name>

# Get cluster version
kubectl version

# Get cluster resources
kubectl get all -A

# Get events
kubectl get events -A --sort-by='.lastTimestamp'
```

### Node Management
```bash
# List nodes
kubectl get nodes

# Label nodes
kubectl label node <node-name> node-role.kubernetes.io/worker=
kubectl label node <node-name> environment=production

# Taint nodes
kubectl taint node <node-name> key=value:NoSchedule
kubectl taint node <node-name> key=value:NoExecute

# Remove taint
kubectl taint node <node-name> key-

# Cordon/uncordon nodes
kubectl cordon <node-name>
kubectl uncordon <node-name>

# Drain node
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

## Networking

### CNI Configuration
```bash
# Default CNI (Canal)
# Combines Calico and Flannel
# No additional configuration needed

# Custom CNI configuration
# /var/lib/rancher/rke2/server/manifests/rke2-canal-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: canal-config
  namespace: kube-system
data:
  cni_network_config: |
    {
      "name": "k8s-pod-network",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "calico",
          "log_level": "info",
          "datastore_type": "kubernetes",
          "mtu": 1440,
          "ipam": {
            "type": "calico-ipam"
          },
          "policy": {
            "type": "k8s"
          },
          "kubernetes": {
            "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
          }
        },
        {
          "type": "portmap",
          "snat": true,
          "capabilities": {"portMappings": true}
        }
      ]
    }
```

### Load Balancer Setup
```yaml
# External load balancer for HA
# Example: HAProxy configuration
global
    maxconn 4096
    log stdout local0
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    mode tcp
    log global
    option tcplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

frontend rke2_frontend
    bind *:9345
    default_backend rke2_backend

frontend k8s_api_frontend
    bind *:6443
    default_backend k8s_api_backend

backend rke2_backend
    balance roundrobin
    server rke2-server-1 192.168.1.100:9345 check
    server rke2-server-2 192.168.1.101:9345 check
    server rke2-server-3 192.168.1.102:9345 check

backend k8s_api_backend
    balance roundrobin
    server rke2-server-1 192.168.1.100:6443 check
    server rke2-server-2 192.168.1.101:6443 check
    server rke2-server-3 192.168.1.102:6443 check
```

## Security

### CIS Hardening
```bash
# Enable CIS hardening during installation
curl -sfL https://get.rke2.io | INSTALL_RKE2_EXEC="--profile=cis-1.23" sh -

# Or add to config.yaml
profile: cis-1.23

# Verify CIS compliance
kubectl run --rm -it cis-benchmark --image=rancher/security-scan:v0.2.2 \
  --restart=Never -- sh -c "cd /tmp && ./run.sh"
```

### FIPS Compliance
```bash
# Install FIPS-compliant version
curl -sfL https://get.rke2.io | INSTALL_RKE2_METHOD="rpm" sh -

# Verify FIPS mode
cat /proc/sys/crypto/fips_enabled

# Check RKE2 FIPS status
rke2 --version
```

### SELinux Configuration
```bash
# Install SELinux policy
sudo yum install -y container-selinux selinux-policy-base
sudo rpm -i https://rpm.rancher.io/k3s-selinux/latest/k3s-selinux-1.2-2.el8.noarch.rpm

# Enable SELinux
sudo setenforce 1
sudo sed -i 's/^SELINUX=permissive$/SELINUX=enforcing/' /etc/selinux/config

# Add to config.yaml
selinux: true
```

### Network Policies
```yaml
# Example network policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
```

## Backup and Restore

### etcd Snapshots
```bash
# Manual snapshot
sudo rke2 etcd-snapshot save --name manual-snapshot-$(date +%Y%m%d%H%M%S)

# List snapshots
sudo rke2 etcd-snapshot list

# Delete snapshot
sudo rke2 etcd-snapshot delete <snapshot-name>

# Configure automatic snapshots in config.yaml
etcd-snapshot-schedule-cron: "0 */6 * * *"
etcd-snapshot-retention: 10
etcd-snapshot-dir: /var/lib/rancher/rke2/server/db/snapshots
```

### Cluster Restore
```bash
# Stop RKE2 on all nodes
sudo systemctl stop rke2-server
sudo systemctl stop rke2-agent

# Restore from snapshot (on first server node)
sudo rke2 etcd-snapshot restore --name <snapshot-name>

# Start RKE2 server
sudo systemctl start rke2-server

# Remove other server nodes from cluster and rejoin
# Start agent nodes
sudo systemctl start rke2-agent
```

### Backup Configuration
```bash
# Backup important files
tar -czf rke2-backup-$(date +%Y%m%d).tar.gz \
  /etc/rancher/rke2/ \
  /var/lib/rancher/rke2/server/db/snapshots/ \
  /var/lib/rancher/rke2/server/tls/ \
  ~/.kube/config

# Backup script
#!/bin/bash
BACKUP_DIR="/backup/rke2"
DATE=$(date +%Y%m%d%H%M%S)
SNAPSHOT_NAME="backup-${DATE}"

# Create snapshot
sudo rke2 etcd-snapshot save --name "${SNAPSHOT_NAME}"

# Create backup directory
mkdir -p "${BACKUP_DIR}/${DATE}"

# Copy files
cp -r /etc/rancher/rke2/ "${BACKUP_DIR}/${DATE}/"
cp -r /var/lib/rancher/rke2/server/db/snapshots/ "${BACKUP_DIR}/${DATE}/"
cp -r /var/lib/rancher/rke2/server/tls/ "${BACKUP_DIR}/${DATE}/"

echo "Backup completed: ${BACKUP_DIR}/${DATE}"
```

## Upgrades

### Upgrade RKE2
```bash
# Check current version
rke2 --version

# Download specific version
curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION="v1.28.4+rke2r1" sh -

# Rolling upgrade process:
# 1. Upgrade server nodes one by one
# 2. Upgrade agent nodes

# Upgrade server node
sudo systemctl stop rke2-server
# Install new version
sudo systemctl start rke2-server

# Verify upgrade
kubectl get nodes
```

### Automated Upgrade with System Upgrade Controller
```yaml
# Install System Upgrade Controller
kubectl apply -f https://github.com/rancher/system-upgrade-controller/releases/latest/download/system-upgrade-controller.yaml

# Create upgrade plan
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: server-plan
  namespace: system-upgrade
spec:
  concurrency: 1
  cordon: true
  nodeSelector:
    matchExpressions:
    - key: node-role.kubernetes.io/control-plane
      operator: In
      values:
      - "true"
  serviceAccountName: system-upgrade
  upgrade:
    image: rancher/rke2-upgrade:v1.28.4-rke2r1
  version: v1.28.4+rke2r1
---
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: agent-plan
  namespace: system-upgrade
spec:
  concurrency: 1
  cordon: true
  nodeSelector:
    matchExpressions:
    - key: node-role.kubernetes.io/control-plane
      operator: DoesNotExist
  prepare:
    args:
    - prepare
    - server-plan
    image: rancher/rke2-upgrade:v1.28.4-rke2r1
  serviceAccountName: system-upgrade
  upgrade:
    image: rancher/rke2-upgrade:v1.28.4-rke2r1
  version: v1.28.4+rke2r1
```

## Monitoring and Logging

### System Monitoring
```bash
# Check service status
sudo systemctl status rke2-server
sudo systemctl status rke2-agent

# View logs
sudo journalctl -u rke2-server -f
sudo journalctl -u rke2-agent -f

# Check containerd
sudo systemctl status rke2-server-containerd
sudo ctr --address /run/k3s/containerd/containerd.sock container list

# Resource usage
top
htop
iostat
df -h
```

### Kubernetes Monitoring
```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Get resource usage
kubectl top nodes
kubectl top pods -A

# Check cluster health
kubectl get componentstatuses
kubectl get --raw='/readyz?verbose'

# Monitor events
kubectl get events -A --sort-by='.lastTimestamp' -w
```

### Prometheus Setup
```yaml
# Prometheus monitoring stack
kubectl create namespace monitoring

# Add Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install kube-prometheus-stack
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false

# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

## Troubleshooting

### Common Issues
```bash
# Node not joining cluster
# Check token and server URL
sudo cat /var/lib/rancher/rke2/server/node-token

# Check connectivity
telnet <server-ip> 9345

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L

# Check SELinux context
ls -laZ /var/lib/rancher/rke2/

# Pod stuck in Pending
kubectl describe pod <pod-name>
kubectl get events

# DNS issues
nslookup kubernetes.default.svc.cluster.local 10.43.0.10
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default

# Network connectivity
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- bash
```

### Log Analysis
```bash
# RKE2 server logs
sudo journalctl -u rke2-server --since "1 hour ago"

# RKE2 agent logs
sudo journalctl -u rke2-agent --since "1 hour ago"

# Containerd logs
sudo journalctl -u rke2-server-containerd --since "1 hour ago"

# Kubernetes component logs
kubectl logs -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l app=traefik

# Node logs
sudo journalctl -u kubelet --since "1 hour ago"
```

### Debug Commands
```bash
# Check cluster certificates
openssl x509 -in /var/lib/rancher/rke2/server/tls/server-ca.crt -text -noout

# Check etcd health
CACERT=/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt
CERT=/var/lib/rancher/rke2/server/tls/etcd/server-client.crt
KEY=/var/lib/rancher/rke2/server/tls/etcd/server-client.key

sudo ETCDCTL_API=3 etcdctl \
  --cacert=${CACERT} \
  --cert=${CERT} \
  --key=${KEY} \
  --endpoints=https://127.0.0.1:2379 \
  endpoint health

# Check API server
curl -k https://localhost:6443/version

# Check node kubelet
sudo /var/lib/rancher/rke2/bin/kubelet --version
```

## Air-gapped Installation

### Prepare Air-gapped Environment
```bash
# Download RKE2 artifacts
mkdir -p rke2-artifacts
cd rke2-artifacts

# Download installer
curl -sfL https://get.rke2.io --output install.sh

# Download binary
wget https://github.com/rancher/rke2/releases/download/v1.28.4%2Brke2r1/rke2.linux-amd64.tar.gz

# Download images
wget https://github.com/rancher/rke2/releases/download/v1.28.4%2Brke2r1/rke2-images.linux-amd64.tar.zst

# Copy to air-gapped environment
scp -r rke2-artifacts/ user@airgapped-server:/tmp/
```

### Install in Air-gapped Environment
```bash
# Install RKE2
INSTALL_RKE2_ARTIFACT_PATH=/tmp/rke2-artifacts sh install.sh

# Load images
sudo mkdir -p /var/lib/rancher/rke2/agent/images/
sudo cp rke2-images.linux-amd64.tar.zst /var/lib/rancher/rke2/agent/images/

# Configure private registry
sudo mkdir -p /etc/rancher/rke2
sudo tee /etc/rancher/rke2/registries.yaml > /dev/null <<EOF
mirrors:
  docker.io:
    endpoint:
      - "https://registry.example.com"
  registry.k8s.io:
    endpoint:
      - "https://registry.example.com"
configs:
  "registry.example.com":
    auth:
      username: admin
      password: password
    tls:
      insecure_skip_verify: true
EOF
```

## Private Registry Configuration

### Registry Configuration
```yaml
# /etc/rancher/rke2/registries.yaml
mirrors:
  docker.io:
    endpoint:
      - "https://registry.example.com"
  registry.k8s.io:
    endpoint:
      - "https://registry.example.com"
  quay.io:
    endpoint:
      - "https://registry.example.com"

configs:
  "registry.example.com":
    auth:
      username: myuser
      password: mypassword
    tls:
      cert_file: /etc/ssl/certs/registry.crt
      key_file: /etc/ssl/private/registry.key
      ca_file: /etc/ssl/certs/registry-ca.crt
      insecure_skip_verify: false
```

## Useful Commands Reference

| Command | Description |
|---------|-------------|
| `sudo systemctl status rke2-server` | Check server status |
| `sudo systemctl status rke2-agent` | Check agent status |
| `sudo journalctl -u rke2-server -f` | View server logs |
| `rke2 etcd-snapshot save` | Create etcd snapshot |
| `rke2 etcd-snapshot list` | List etcd snapshots |
| `kubectl get nodes` | List cluster nodes |
| `kubectl get pods -A` | List all pods |
| `kubectl describe node <name>` | Describe node |
| `kubectl drain <node>` | Drain node |
| `kubectl cordon <node>` | Cordon node |

## Configuration Files

### Important File Locations
```bash
# Configuration
/etc/rancher/rke2/config.yaml
/etc/rancher/rke2/registries.yaml

# Kubeconfig
/etc/rancher/rke2/rke2.yaml

# Data directory
/var/lib/rancher/rke2/

# Server data
/var/lib/rancher/rke2/server/

# Agent data
/var/lib/rancher/rke2/agent/

# etcd snapshots
/var/lib/rancher/rke2/server/db/snapshots/

# TLS certificates
/var/lib/rancher/rke2/server/tls/

# Binaries
/var/lib/rancher/rke2/bin/

# Node token
/var/lib/rancher/rke2/server/node-token

# Service files
/etc/systemd/system/rke2-server.service
/etc/systemd/system/rke2-agent.service
```

## Environment Variables

```bash
# RKE2 installation
export INSTALL_RKE2_VERSION="v1.28.4+rke2r1"
export INSTALL_RKE2_TYPE="server"  # or "agent"
export INSTALL_RKE2_METHOD="rpm"   # or "tar"
export INSTALL_RKE2_ARTIFACT_PATH="/tmp/rke2-artifacts"

# Kubernetes configuration
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export PATH=$PATH:/var/lib/rancher/rke2/bin

# Containerd
export CONTAINERD_ADDRESS=/run/k3s/containerd/containerd.sock
```

## Best Practices

### Production Recommendations
1. **High Availability** - Use 3 or 5 server nodes
2. **etcd Backup** - Configure automatic snapshots
3. **Load Balancer** - Use external load balancer for API server
4. **Security** - Enable CIS hardening and FIPS compliance
5. **Monitoring** - Deploy comprehensive monitoring stack
6. **Resource Planning** - Size nodes appropriately
7. **Network Policies** - Implement network segmentation
8. **Registry** - Use private registry for air-gapped environments
9. **Upgrades** - Plan and test upgrade procedures
10. **Documentation** - Maintain cluster documentation

---

*For more detailed information, visit the [official RKE2 documentation](https://docs.rke2.io/)*

