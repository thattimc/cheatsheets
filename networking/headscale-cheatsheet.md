# Headscale Cheatsheet

Headscale is an open-source, self-hosted implementation of the Tailscale control server.
It allows you to run your own Tailscale network without relying on Tailscale's SaaS offering.

## Installation

### Docker (Recommended)

```bash
# Pull the latest image
docker pull headscale/headscale:latest

# Create config directory
mkdir -p ./config

# Run Headscale
docker run \
  --name headscale \
  --detach \
  --volume $(pwd)/config:/etc/headscale \
  --publish 8080:8080 \
  --publish 9090:9090 \
  headscale/headscale:latest headscale serve
```

### Binary Installation (Linux)

```bash
# Download latest release
wget https://github.com/juanfont/headscale/releases/latest/download/headscale_linux_amd64
chmod +x headscale_linux_amd64
sudo mv headscale_linux_amd64 /usr/local/bin/headscale
```

### From Source

```bash
git clone https://github.com/juanfont/headscale.git
cd headscale
go build -ldflags "-s -w -X github.com/juanfont/headscale/cmd/headscale/cli.Version=$(git describe --tags)" ./cmd/headscale
```

## Configuration

### Basic Configuration File

```yaml
# /etc/headscale/config.yaml
server_url: https://headscale.example.com
listen_addr: 0.0.0.0:8080
metrics_listen_addr: 127.0.0.1:9090
grpc_listen_addr: 0.0.0.0:50443
grpc_allow_insecure: false

private_key_path: /etc/headscale/private.key
noise:
  private_key_path: /etc/headscale/noise_private.key

ip_prefixes:
  - fd7a:115c:a1e0::/48
  - 100.64.0.0/10

derp:
  server:
    enabled: false
  urls:
    - https://controlplane.tailscale.com/derpmap/default

database:
  type: sqlite3
  sqlite:
    path: /etc/headscale/db.sqlite

log:
  level: info

dns_config:
  override_local_dns: true
  nameservers:
    - 1.1.1.1
    - 8.8.8.8
  domains: []
  magic_dns: true
  base_domain: example.com
```

### Generate Private Keys

```bash
# Generate private key
headscale generate private-key > /etc/headscale/private.key

# Generate noise private key
headscale generate noise-key > /etc/headscale/noise_private.key
```

## Server Management

### Start/Stop Server

```bash
# Start server
headscale serve

# Start with specific config
headscale --config /path/to/config.yaml serve

# Run in background
nohup headscale serve > headscale.log 2>&1 &
```

### Systemd Service

```bash
# Create systemd service file
sudo tee /etc/systemd/system/headscale.service > /dev/null <<EOF
[Unit]
Description=headscale controller
After=syslog.target
After=network.target

[Service]
Type=simple
User=headscale
Group=headscale
ExecStart=/usr/local/bin/headscale serve
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable headscale
sudo systemctl start headscale
```

## User Management

### Create Users

```bash
# Create a new user
headscale users create myuser

# List all users
headscale users list

# Delete a user
headscale users delete myuser

# Rename a user
headscale users rename oldname newname
```

## Pre-auth Keys

### Generate Pre-auth Keys

```bash
# Create a pre-auth key
headscale preauthkeys create --user myuser

# Create with expiration
headscale preauthkeys create --user myuser --expiration 24h

# Create reusable key
headscale preauthkeys create --user myuser --reusable

# Create ephemeral key (node deleted when disconnected)
headscale preauthkeys create --user myuser --ephemeral

# List pre-auth keys
headscale preauthkeys list

# Expire a pre-auth key
headscale preauthkeys expire --identifier <key-id>
```

## Node Management

### Register Nodes

```bash
# Register a node manually
headscale nodes register --user myuser --key <machine-key>

# List all nodes
headscale nodes list

# List nodes for specific user
headscale nodes list --user myuser

# Show node details
headscale nodes show <node-id>
```

### Node Operations

```bash
# Delete a node
headscale nodes delete <node-id>

# Move node to different user
headscale nodes move <node-id> --user newuser

# Set node tags
headscale nodes tag <node-id> --tags tag1,tag2

# Rename a node
headscale nodes rename <node-id> --name new-name

# Expire node key
headscale nodes expire <node-id>
```

## Route Management

### Subnet Routes

```bash
# Enable route for node
headscale routes enable <route-id>

# Disable route
headscale routes disable <route-id>

# List all routes
headscale routes list

# List routes for specific node
headscale routes list --node <node-id>

# Delete a route
headscale routes delete <route-id>
```

### Exit Nodes

```bash
# Enable node as exit node
headscale routes enable <route-id>  # where route is 0.0.0.0/0

# List exit nodes
headscale routes list | grep "0.0.0.0/0"
```

## ACL (Access Control Lists)

### ACL Configuration

```bash
# Set ACL policy from file
headscale acl set policy.json

# Get current ACL policy
headscale acl get

# Validate ACL policy
headscale acl validate policy.json
```

### Sample ACL Policy

```json
{
  "hosts": {
    "office": "100.64.0.1",
    "home": "100.64.0.2"
  },
  "groups": {
    "group:admin": ["admin@example.com"],
    "group:dev": ["dev1@example.com", "dev2@example.com"]
  },
  "tagOwners": {
    "tag:server": ["group:admin"],
    "tag:client": ["group:dev"]
  },
  "acls": [
    {
      "action": "accept",
      "src": ["group:admin"],
      "dst": ["*:*"]
    },
    {
      "action": "accept",
      "src": ["group:dev"],
      "dst": ["tag:server:22", "tag:server:80", "tag:server:443"]
    }
  ]
}
```

## API Keys

### Generate API Keys

```bash
# Create API key
headscale apikeys create

# Create with expiration
headscale apikeys create --expiration 30d

# List API keys
headscale apikeys list

# Expire API key
headscale apikeys expire <prefix>
```

### Using API Keys

```bash
# Set API key as environment variable
export HEADSCALE_API_KEY="your-api-key-here"

# Use with curl
curl -H "Authorization: Bearer $HEADSCALE_API_KEY" \
     https://headscale.example.com/api/v1/users
```

## Client Configuration

### Connect Tailscale Client to Headscale

```bash
# Set custom control server
tailscale up --login-server https://headscale.example.com

# Use pre-auth key
tailscale up --login-server https://headscale.example.com --authkey <pre-auth-key>

# Accept routes and DNS
tailscale up --login-server https://headscale.example.com --accept-routes --accept-dns
```

### Reset Client

```bash
# Reset tailscale client
tailscale logout
sudo tailscale down
sudo rm -rf /var/lib/tailscale/
tailscale up --login-server https://headscale.example.com
```

## Database Management

### SQLite Operations

```bash
# Backup database
sqlite3 /etc/headscale/db.sqlite ".backup /path/to/backup.db"

# Restore database
sqlite3 /etc/headscale/db.sqlite ".restore /path/to/backup.db"

# View tables
sqlite3 /etc/headscale/db.sqlite ".tables"
```

### PostgreSQL Configuration

```yaml
# In config.yaml
database:
  type: postgres
  postgres:
    host: localhost
    port: 5432
    name: headscale
    user: headscale
    pass: password
    ssl_mode: disable
```

## Monitoring & Logging

### View Logs

```bash
# View logs (systemd)
sudo journalctl -u headscale -f

# View logs (docker)
docker logs -f headscale

# Debug level logging
headscale --log-level debug serve
```

### Metrics

```bash
# Access Prometheus metrics
curl http://localhost:9090/metrics

# Health check
curl http://localhost:8080/health
```

## Troubleshooting

### Common Issues

```bash
# Check server status
headscale debug create-node-key

# Verify configuration
headscale config validate

# Test connectivity
telnet headscale.example.com 8080

# Check DNS resolution
nslookup headscale.example.com

# Verify certificates (if using HTTPS)
openssl s_client -connect headscale.example.com:443
```

### Node Registration Issues

```bash
# Manual node registration
# 1. On client, get machine key
tailscale up --login-server https://headscale.example.com
# 2. Copy the machine key from error/logs
# 3. Register on server
headscale nodes register --user myuser --key mkey:...
```

### Certificate Issues

```bash
# Generate self-signed certificate
openssl req -new -x509 -key private.key -out cert.crt -days 365

# Use Let's Encrypt with certbot
certbot --nginx -d headscale.example.com
```

## Useful Commands Reference

| Command Category | Command | Description |
|---|---|---|
| **Users** | `headscale users create <name>` | Create new user |
| | `headscale users list` | List all users |
| | `headscale users delete <name>` | Delete user |
| **Nodes** | `headscale nodes list` | List all nodes |
| | `headscale nodes delete <id>` | Delete node |
| | `headscale nodes move <id> --user <user>` | Move node to user |
| **PreAuth** | `headscale preauthkeys create --user <user>` | Create pre-auth key |
| | `headscale preauthkeys list` | List pre-auth keys |
| **Routes** | `headscale routes list` | List all routes |
| | `headscale routes enable <id>` | Enable route |
| **ACL** | `headscale acl set <file>` | Set ACL policy |
| | `headscale acl get` | Get current ACL |
| **API Keys** | `headscale apikeys create` | Create API key |
| | `headscale apikeys list` | List API keys |

## Docker Compose Example

```yaml
# docker-compose.yml
version: '3.8'
services:
  headscale:
    image: headscale/headscale:latest
    container_name: headscale
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "9090:9090"
    volumes:
      - ./config:/etc/headscale
      - ./data:/var/lib/headscale
    command: headscale serve
    environment:
      - TZ=UTC
```

## Security Best Practices

1. **Use HTTPS**: Always run Headscale behind HTTPS with valid certificates
2. **Firewall**: Restrict access to Headscale ports (8080, 9090)
3. **Pre-auth Keys**: Use ephemeral and time-limited pre-auth keys
4. **ACLs**: Implement proper access control policies
5. **API Keys**: Rotate API keys regularly
6. **Database**: Secure your database with proper authentication
7. **Updates**: Keep Headscale updated to latest version

## Migration from Tailscale

```bash
# 1. Export Tailscale state (if possible)
# 2. Reset Tailscale clients
tailscale logout
sudo tailscale down
sudo rm -rf /var/lib/tailscale/

# 3. Connect to Headscale
tailscale up --login-server https://headscale.example.com
```

---

*For more detailed information, visit the [official Headscale documentation](https://headscale.net/) and [GitHub repository](https://github.com/juanfont/headscale)*
