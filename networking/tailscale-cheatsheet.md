# Tailscale Cheatsheet

Tailscale is a zero-config VPN that creates a secure network between your devices using WireGuard.

## Installation

### macOS

```bash
brew install tailscale
# or download from https://tailscale.com/download
```

### Linux (Ubuntu/Debian)

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

### Windows

```powershell
# Download from https://tailscale.com/download
# or use winget
winget install tailscale.tailscale
```

## Basic Commands

### Authentication & Setup

```bash
# Start Tailscale and authenticate
tailscale up

# Login with specific flags
tailscale up --accept-routes --accept-dns

# Logout
tailscale logout

# Check authentication status
tailscale status
```

### Network Management

```bash
# Show network status
tailscale status

# Show detailed status
tailscale status --json

# Show only IP addresses
tailscale ip

# Show IPv4 only
tailscale ip -4

# Show IPv6 only
tailscale ip -6
```

### Connection Control

```bash
# Connect to Tailscale network
tailscale up

# Disconnect from network (but stay authenticated)
tailscale down

# Restart connection
tailscale up --reset
```

## Advanced Features

### Exit Nodes

```bash
# Advertise as exit node
tailscale up --advertise-exit-node

# Use specific exit node
tailscale up --exit-node=<node-ip>

# Stop using exit node
tailscale up --exit-node=""

# List available exit nodes
tailscale exit-node list
```

### Subnet Routing

```bash
# Advertise subnet routes
tailscale up --advertise-routes=192.168.1.0/24,10.0.0.0/8

# Accept subnet routes from other nodes
tailscale up --accept-routes
```

### DNS Configuration

```bash
# Accept DNS configuration
tailscale up --accept-dns

# Set custom DNS
tailscale up --accept-dns=false
```

### SSH Integration

```bash
# Enable Tailscale SSH
tailscale up --ssh

# SSH to another node
ssh user@<tailscale-hostname>

# SSH using Tailscale IP
ssh user@<tailscale-ip>
```

## File Sharing (Taildrop)

```bash
# Send file to another device
tailscale file cp myfile.txt <hostname>:

# Send to specific user on device
tailscale file cp document.pdf <hostname>:<username>

# Get received files (usually in ~/Downloads)
tailscale file get
```

## Network Diagnostics

### Ping and Connectivity

```bash
# Ping another Tailscale node
tailscale ping <hostname-or-ip>

# Detailed network path info
tailscale netcheck

# Show network map
tailscale status --peers
```

### Debugging

```bash
# Show detailed logs
tailscale debug

# Show network interface info
ip addr show tailscale0  # Linux
ifconfig utun3          # macOS

# Show routing table
tailscale status --json | jq '.Peer[].TailscaleIPs'
```

## Device Management

### Device Information

```bash
# Show current device info
tailscale whois $(tailscale ip -4)

# Show info about another device
tailscale whois <ip-or-hostname>

# List all devices
tailscale status --peers
```

### Device Control

```bash
# Set device hostname
tailscale up --hostname=<new-hostname>

# Disable key expiry
tailscale up --auth-key=<auth-key>

# Force re-authentication
tailscale up --force-reauth
```

## Security & Access Control

### MagicDNS

```bash
# Enable MagicDNS (usually automatic)
tailscale up --accept-dns

# Access devices by name
ping <device-name>.your-tailnet.ts.net
ssh user@<device-name>
```

### Tailscale Lock (Network Lock)

```bash
# Initialize Tailscale Lock
tailscale lock init

# Add signing key
tailscale lock add <key-id>

# Show lock status
tailscale lock status
```

## Configuration Files

### Service Configuration

```bash
# Linux systemd service
sudo systemctl status tailscaled
sudo systemctl restart tailscaled

# macOS launchd service  
sudo launchctl list | grep tailscale
sudo launchctl unload /Library/LaunchDaemons/com.tailscale.tailscaled.plist
```

### Config Locations

- **Linux**: `/var/lib/tailscale/`
- **macOS**: `/Library/Tailscale/`
- **Windows**: `%ProgramData%\Tailscale\`

## Common Use Cases

### Remote Access Setup

```bash
# 1. Install on remote server
sudo tailscale up --ssh --advertise-exit-node

# 2. Install on local machine
tailscale up --accept-routes

# 3. SSH to remote server
ssh user@<server-tailscale-name>
```

### Site-to-Site VPN

```bash
# On site router/gateway
tailscale up --advertise-routes=192.168.1.0/24 --accept-routes

# On other site
tailscale up --advertise-routes=10.0.0.0/24 --accept-routes
```

### Secure Internet Access

```bash
# Set up exit node
tailscale up --advertise-exit-node

# Use exit node from client
tailscale up --exit-node=<exit-node-ip>
```

## Useful Flags & Options

| Flag | Description |
|------|-------------|
| `--accept-dns` | Accept DNS configuration from Tailscale |
| `--accept-routes` | Accept subnet routes from other nodes |
| `--advertise-exit-node` | Advertise as exit node |
| `--advertise-routes` | Advertise specific subnet routes |
| `--exit-node` | Use specific exit node |
| `--hostname` | Set device hostname |
| `--shields-up` | Block incoming connections |
| `--ssh` | Enable Tailscale SSH |
| `--reset` | Reset connection state |
| `--auth-key` | Use auth key for unattended setup |

## Troubleshooting

### Common Issues

```bash
# Connection problems
tailscale netcheck
tailscale ping <target>

# DNS issues
nslookup <hostname>.your-tailnet.ts.net

# Firewall issues
sudo ufw allow in on tailscale0

# Service issues
sudo systemctl status tailscaled
sudo journalctl -u tailscaled -f
```

### Reset & Clean Start

```bash
# Complete reset
tailscale logout
sudo rm -rf /var/lib/tailscale/  # Linux
tailscale up
```

## Web Interface

- **Admin Console**: <https://login.tailscale.com/admin/>
- **Device Management**: Manage devices, users, and settings
- **Access Controls**: Set up ACLs and device policies
- **DNS Settings**: Configure MagicDNS and nameservers

## Pro Tips

1. **Use MagicDNS**: Access devices by name instead of IP
2. **Set up Exit Nodes**: Route internet traffic through trusted devices
3. **Enable SSH**: Secure SSH access without port forwarding
4. **Use Subnet Routes**: Connect entire networks
5. **Taildrop**: Easy file sharing between devices
6. **Mobile Apps**: Install on phones/tablets for full network access

---

*For more detailed information, visit the [official Tailscale documentation](https://tailscale.com/kb/)*
