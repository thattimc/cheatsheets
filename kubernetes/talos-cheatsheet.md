# Talos Linux Cheatsheet

## Installation & Setup

### Install talosctl
```bash
# macOS
brew install siderolabs/tap/talosctl

# Linux
curl -sL https://talos.dev/install | sh

# Windows
choco install talosctl

# Direct download
curl -Lo /usr/local/bin/talosctl https://github.com/siderolabs/talos/releases/latest/download/talosctl-$(uname -s | tr "[:upper:]" "[:lower:]")-amd64
chmod +x /usr/local/bin/talosctl
```

### Generate Configuration
```bash
# Generate cluster configuration
talosctl gen config cluster-name https://cluster-endpoint:6443

# Generate with custom options
talosctl gen config cluster-name https://cluster-endpoint:6443 \
  --config-patch-control-plane @patch-control-plane.yaml \
  --config-patch-worker @patch-worker.yaml

# Generate with additional SANs
talosctl gen config cluster-name https://cluster-endpoint:6443 \
  --additional-sans 192.168.1.100,cluster.local
```

## Cluster Management

### Bootstrap & Apply Configuration
```bash
# Apply configuration to node
talosctl apply-config --insecure --nodes 192.168.1.10 --file controlplane.yaml

# Apply to worker nodes
talosctl apply-config --insecure --nodes 192.168.1.20,192.168.1.21 --file worker.yaml

# Bootstrap etcd on first control plane
talosctl bootstrap --nodes 192.168.1.10 --endpoints 192.168.1.10

# Get kubeconfig
talosctl kubeconfig --nodes 192.168.1.10 --endpoints 192.168.1.10
```

### Node Operations
```bash
# Reboot node
talosctl reboot --nodes 192.168.1.10

# Shutdown node
talosctl shutdown --nodes 192.168.1.10

# Reset node (DESTRUCTIVE)
talosctl reset --nodes 192.168.1.10

# Upgrade node
talosctl upgrade --nodes 192.168.1.10 --image ghcr.io/siderolabs/talos:v1.5.0

# Upgrade Kubernetes
talosctl upgrade-k8s --nodes 192.168.1.10 --to 1.28.0
```

## Configuration Management

### View & Edit Configuration
```bash
# Get current configuration
talosctl get machineconfig --nodes 192.168.1.10

# Edit configuration
talosctl edit machineconfig --nodes 192.168.1.10

# Patch configuration
talosctl patch machineconfig --nodes 192.168.1.10 --patch @patch.yaml

# Apply new configuration
talosctl apply-config --nodes 192.168.1.10 --file new-config.yaml
```

### Configuration Patches
```yaml
# Example patch file (patch.yaml)
- op: replace
  path: /machine/time/servers
  value:
    - time.cloudflare.com
    - pool.ntp.org

- op: add
  path: /machine/network/hostname
  value: node-01
```

## Monitoring & Diagnostics

### System Information
```bash
# Get node health
talosctl health --nodes 192.168.1.10

# Get system version
talosctl version --nodes 192.168.1.10

# Get cluster health
talosctl health --control-plane-nodes 192.168.1.10,192.168.1.11,192.168.1.12

# Get node stats
talosctl stats --nodes 192.168.1.10
```

### Logs & Debugging
```bash
# View system logs
talosctl logs --nodes 192.168.1.10

# View specific service logs
talosctl logs --nodes 192.168.1.10 kubelet
talosctl logs --nodes 192.168.1.10 etcd
talosctl logs --nodes 192.168.1.10 containerd

# Follow logs in real-time
talosctl logs --nodes 192.168.1.10 --follow

# View kernel logs
talosctl dmesg --nodes 192.168.1.10
```

### System Processes
```bash
# List running processes
talosctl ps --nodes 192.168.1.10

# List system services
talosctl services --nodes 192.168.1.10

# Get service status
talosctl service kubelet --nodes 192.168.1.10

# Restart service
talosctl service kubelet restart --nodes 192.168.1.10
```

## Resource Management

### Talos Resources
```bash
# List all resources
talosctl get all --nodes 192.168.1.10

# Get specific resource types
talosctl get nodes --nodes 192.168.1.10
talosctl get pods --nodes 192.168.1.10
talosctl get services --nodes 192.168.1.10

# Get resource with output format
talosctl get nodes -o yaml --nodes 192.168.1.10
talosctl get nodes -o json --nodes 192.168.1.10
```

### Disk & Storage
```bash
# List disks
talosctl get disks --nodes 192.168.1.10

# Get disk usage
talosctl df --nodes 192.168.1.10

# List mounts
talosctl get mounts --nodes 192.168.1.10
```

### Network
```bash
# Get network interfaces
talosctl get addresses --nodes 192.168.1.10
talosctl get routes --nodes 192.168.1.10

# Get network links
talosctl get links --nodes 192.168.1.10

# Get hostname
talosctl get hostname --nodes 192.168.1.10
```

## Container Management

### Container Operations
```bash
# List containers
talosctl containers --nodes 192.168.1.10

# List containers with Kubernetes info
talosctl containers --kubernetes --nodes 192.168.1.10

# Get container logs
talosctl logs --nodes 192.168.1.10 --container container-name

# Execute command in container
talosctl exec --nodes 192.168.1.10 --container container-name -- /bin/sh
```

### Image Management
```bash
# List images
talosctl images --nodes 192.168.1.10

# Pull image
talosctl pull --nodes 192.168.1.10 nginx:latest

# Remove unused images
talosctl image prune --nodes 192.168.1.10
```

## Security & Certificates

### Certificate Management
```bash
# Get cluster certificates
talosctl get certificates --nodes 192.168.1.10

# Get etcd certificates
talosctl get etcdcerts --nodes 192.168.1.10

# Rotate certificates
talosctl rotate-ca --nodes 192.168.1.10
```

### Security Scanning
```bash
# Run security scan
talosctl conformance --nodes 192.168.1.10

# Get security profile
talosctl get securityprofile --nodes 192.168.1.10
```

## Maintenance & Backup

### Etcd Operations
```bash
# Create etcd snapshot
talosctl etcd snapshot --nodes 192.168.1.10

# List etcd members
talosctl etcd members --nodes 192.168.1.10

# Remove etcd member
talosctl etcd remove-member --nodes 192.168.1.10 --member member-id

# Etcd status
talosctl etcd status --nodes 192.168.1.10
```

### Backup & Recovery
```bash
# Create cluster backup
talosctl etcd snapshot --nodes 192.168.1.10 --output backup.db

# Restore from backup (during bootstrap)
talosctl bootstrap --nodes 192.168.1.10 --recover-from backup.db
```

## Troubleshooting

### Common Issues
```bash
# Check node readiness
talosctl health --nodes 192.168.1.10

# Verify network connectivity
talosctl ping --nodes 192.168.1.10

# Check time synchronization
talosctl get time --nodes 192.168.1.10

# Validate configuration
talosctl validate --config controlplane.yaml
```

### Recovery Operations
```bash
# Force reboot if node is unresponsive
talosctl reboot --nodes 192.168.1.10 --force

# Emergency shell access
talosctl shell --nodes 192.168.1.10

# Copy files to/from node
talosctl copy --nodes 192.168.1.10 /local/file /remote/path
talosctl copy --nodes 192.168.1.10 --to-local /remote/file /local/path
```

## Configuration Examples

### Basic Control Plane Configuration
```yaml
version: v1alpha1
kind: MachineConfig
machine:
  type: controlplane
  token: your-token-here
  ca:
    cert: LS0tLS1CRUdJTi...
    key: LS0tLS1CRUdJTi...
  certSANs:
    - 192.168.1.10
    - cluster.local
  kubelet:
    image: ghcr.io/siderolabs/kubelet:v1.28.0
  network:
    hostname: control-plane-01
    interfaces:
      - interface: eth0
        dhcp: true
cluster:
  name: my-cluster
  controlPlane:
    endpoint: https://192.168.1.10:6443
  network:
    dnsDomain: cluster.local
    podSubnets:
      - 10.244.0.0/16
    serviceSubnets:
      - 10.96.0.0/12
```

### Basic Worker Configuration
```yaml
version: v1alpha1
kind: MachineConfig
machine:
  type: worker
  token: your-token-here
  ca:
    cert: LS0tLS1CRUdJTi...
  kubelet:
    image: ghcr.io/siderolabs/kubelet:v1.28.0
  network:
    hostname: worker-01
    interfaces:
      - interface: eth0
        dhcp: true
cluster:
  name: my-cluster
  controlPlane:
    endpoint: https://192.168.1.10:6443
```

## Advanced Features

### Custom Registry Configuration
```yaml
machine:
  registries:
    mirrors:
      docker.io:
        endpoints:
          - https://registry.local:5000
    config:
      registry.local:5000:
        tls:
          insecureSkipVerify: true
        auth:
          username: user
          password: pass
```

### Disk Encryption
```yaml
machine:
  systemDiskEncryption:
    ephemeral:
      provider: luks2
      keys:
        - static:
            passphrase: your-passphrase
```

### Custom Kernel Parameters
```yaml
machine:
  kernel:
    modules:
      - name: br_netfilter
        parameters:
          - nf_conntrack_max=131072
```

## Useful Commands

### Quick Status Check
```bash
# One-liner cluster health check
talosctl health --control-plane-nodes $(talosctl config info | grep endpoints | cut -d: -f2 | tr -d ' ')

# Get all node IPs
talosctl get nodes -o json | jq -r '.spec.addresses[] | select(.type=="InternalIP") | .address'

# Check Kubernetes version across nodes
talosctl version --short --nodes 192.168.1.10,192.168.1.11,192.168.1.12
```

### Automation Scripts
```bash
#!/bin/bash
# Bulk node operations
NODES="192.168.1.10,192.168.1.11,192.168.1.12"

# Apply configuration to all nodes
for config in controlplane.yaml worker.yaml; do
  talosctl apply-config --insecure --nodes $NODES --file $config
done

# Wait for nodes to be ready
talosctl health --wait-timeout 10m --nodes $NODES
```

## Configuration Management

### Talosctl Config
```bash
# List contexts
talosctl config contexts

# Set context
talosctl config context cluster-name

# Add new context
talosctl config add cluster-name --endpoints 192.168.1.10 --nodes 192.168.1.10,192.168.1.11

# Remove context
talosctl config remove cluster-name
```

### Environment Variables
```bash
# Set default endpoints
export TALOSCONFIG="~/.talos/config"
export TALOS_ENDPOINTS="192.168.1.10,192.168.1.11,192.168.1.12"

# Set default nodes
export TALOS_NODES="192.168.1.10,192.168.1.11,192.168.1.12"
```

## Performance Tuning

### Resource Limits
```yaml
machine:
  kubelet:
    extraArgs:
      max-pods: 250
      kube-reserved: cpu=100m,memory=100Mi
      system-reserved: cpu=100m,memory=100Mi
```

### Network Optimization
```yaml
machine:
  network:
    interfaces:
      - interface: eth0
        mtu: 9000  # Jumbo frames
        dhcp: true
```

## Tips & Best Practices

### Security
1. **Always use TLS** for cluster communication
2. **Rotate certificates regularly** using `talosctl rotate-ca`
3. **Use disk encryption** for sensitive workloads
4. **Limit SSH access** - Talos doesn't need SSH
5. **Keep Talos updated** with latest security patches

### Operations
1. **Backup etcd regularly** with `talosctl etcd snapshot`
2. **Test configuration changes** on non-production first
3. **Monitor cluster health** with `talosctl health`
4. **Use configuration patches** for small changes
5. **Document your patches** and configurations

### Automation
1. **Use GitOps** for configuration management
2. **Automate upgrades** with proper testing
3. **Monitor logs** for early issue detection
4. **Set up alerts** for cluster health
5. **Use infrastructure as code** for reproducible clusters

