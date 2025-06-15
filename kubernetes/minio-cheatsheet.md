# MinIO Cheatsheet

MinIO is a high-performance, S3-compatible object storage system built for cloud-native
applications. It provides enterprise-grade features like data protection, scalability, and
security while maintaining simplicity and ease of use.

## Overview

### Key Features

- **S3 Compatible** - Full compatibility with Amazon S3 API
- **High Performance** - Sub-millisecond latency and high throughput
- **Distributed** - Horizontally scalable across multiple nodes
- **Erasure Coding** - Data protection with configurable redundancy
- **Multi-Tenancy** - Support for multiple users and access policies
- **Encryption** - Client-side and server-side encryption
- **Cross-Platform** - Runs on any OS, container, or cloud
- **Kubernetes Native** - First-class Kubernetes support
- **Event Notifications** - Real-time event notifications
- **Lifecycle Management** - Automated data lifecycle policies
- **Versioning** - Object versioning support
- **Replication** - Cross-site and cross-cloud replication

### Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Apps  │    │   MinIO Server  │    │   Storage       │
│                 │    │                 │    │                 │
│ • S3 SDKs       │◄──►│ • S3 API        │◄──►│ • Local Disks   │
│ • Web Console   │    │ • Admin API     │    │ • Network Disks │
│ • CLI Tools     │    │ • Gateway       │    │ • Cloud Storage │
│ • Applications  │    │ • Load Balancer │    │ • Erasure Coded │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                       ┌─────────────────┐
                       │   Distributed   │
                       │                 │
                       │ • Multi-Node    │
                       │ • Clustering    │
                       │ • Replication   │
                       │ • Federation    │
                       └─────────────────┘
```

## Installation

### macOS Installation

```bash
# Using Homebrew
brew install minio/stable/minio
brew install minio/stable/mc  # MinIO Client

# Using Binary
wget https://dl.min.io/server/minio/release/darwin-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/

# Install MinIO Client
wget https://dl.min.io/client/mc/release/darwin-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/

# Verify installation
minio --version
mc --version
```

### Linux Installation

```bash
# Download and install MinIO server
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/

# Download and install MinIO client
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/

# Create systemd service
sudo cat > /etc/systemd/system/minio.service << EOF
[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
WorkingDirectory=/usr/local/

User=minio-user
Group=minio-user
ProtectProc=invisible

EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

# Let systemd restart this service always
Restart=always

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536

# Specifies the maximum number of threads this process can create
TasksMax=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
EOF

# Create user
sudo useradd -r minio-user -s /sbin/nologin

# Create configuration file
sudo cat > /etc/default/minio << EOF
# MINIO_ROOT_USER and MINIO_ROOT_PASSWORD sets the root account for the MinIO server.
# This user has unrestricted permissions to perform S3 and administrative API operations on any resource in the deployment.
# Omit to use the default values 'minioadmin:minioadmin'.
MINIO_ROOT_USER=myminioadmin
MINIO_ROOT_PASSWORD=minio-secret-key-change-me

# MINIO_VOLUMES sets the storage volume or path to use for the MinIO server.
MINIO_VOLUMES="/usr/local/share/minio"

# MINIO_OPTS sets any additional commandline options to pass to the MinIO server.
# For example, `--console-address :9001` sets the MinIO Console listen port
MINIO_OPTS="--console-address :9001"
EOF

# Create data directory
sudo mkdir -p /usr/local/share/minio
sudo chown minio-user:minio-user /usr/local/share/minio

# Start and enable service
sudo systemctl enable minio.service
sudo systemctl start minio.service
sudo systemctl status minio.service
```

### Docker Installation

```bash
# Run MinIO server
docker run -p 9000:9000 -p 9001:9001 \
  -e "MINIO_ROOT_USER=minioadmin" \
  -e "MINIO_ROOT_PASSWORD=minioadmin" \
  minio/minio server /data --console-address ":9001"

# Run with persistent storage
docker run -p 9000:9000 -p 9001:9001 \
  -v /mnt/data:/data \
  -e "MINIO_ROOT_USER=minioadmin" \
  -e "MINIO_ROOT_PASSWORD=minioadmin" \
  minio/minio server /data --console-address ":9001"

# Using Docker Compose
cat > docker-compose.yml << EOF
version: '3.7'

services:
  minio:
    image: minio/minio:latest
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - minio_storage:/data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

volumes:
  minio_storage:
EOF

docker-compose up -d

# MinIO Client in Docker
docker run -it --entrypoint=/bin/sh minio/mc
```

### Kubernetes Installation

```yaml
# Using MinIO Operator
kubectl apply -k "github.com/minio/operator?ref=v4.5.8"

# Create namespace
kubectl create namespace minio-operator

# Deploy MinIO Tenant
cat > minio-tenant.yaml << EOF
apiVersion: minio.min.io/v2
kind: Tenant
metadata:
  name: minio
  namespace: minio-tenant
spec:
  image: minio/minio:RELEASE.2023-12-01T01-11-11Z
  credsSecret:
    name: minio-creds-secret
  pools:
    - servers: 4
      name: pool-0
      volumesPerServer: 4
      volumeClaimTemplate:
        metadata:
          name: data
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 1Ti
  mountPath: /export
  subPath: /data
  requestAutoCert: false
EOF

# Create credentials secret
kubectl create secret generic minio-creds-secret \
  --from-literal=accesskey=minio \
  --from-literal=secretkey=minio123 \
  --namespace minio-tenant

kubectl apply -f minio-tenant.yaml
```

## Basic MinIO Server Commands

### Starting MinIO Server

```bash
# Basic server start
minio server /data

# With console address
minio server /data --console-address ":9001"

# Multiple drives (erasure coding)
minio server /data1 /data2 /data3 /data4

# Distributed mode
minio server http://192.168.1.11/export1 http://192.168.1.12/export1 \
             http://192.168.1.13/export1 http://192.168.1.14/export1

# With environment variables
export MINIO_ROOT_USER=minioadmin
export MINIO_ROOT_PASSWORD=minioadmin
minio server /data

# With TLS
minio server /data --certs-dir /etc/minio/certs

# Background mode
nohup minio server /data > minio.log 2>&1 &

# Check server status
curl http://localhost:9000/minio/health/live
```

### Server Configuration

```bash
# Environment variables
export MINIO_ROOT_USER=admin                    # Access key
export MINIO_ROOT_PASSWORD=password123          # Secret key
export MINIO_BROWSER=on                         # Enable web browser
export MINIO_DOMAIN=storage.example.com         # Domain name
export MINIO_SERVER_URL=https://storage.example.com  # Server URL
export MINIO_BROWSER_REDIRECT_URL=https://console.example.com  # Console URL

# Advanced configuration
export MINIO_CACHE_DRIVES="/tmp/drive1,/tmp/drive2"    # Cache drives
export MINIO_CACHE_EXCLUDE="*.pdf,*.doc"              # Cache exclusions
export MINIO_CACHE_QUOTA=80                           # Cache quota percentage
export MINIO_CACHE_AFTER=3                            # Cache after N access
export MINIO_CACHE_WATERMARK_LOW=70                   # Low watermark
export MINIO_CACHE_WATERMARK_HIGH=90                  # High watermark

# Compression
export MINIO_COMPRESS_ENABLE=on
export MINIO_COMPRESS_EXTENSIONS=".pdf,.doc,.txt"
export MINIO_COMPRESS_MIME_TYPES="application/pdf"

# Encryption
export MINIO_KMS_SECRET_KEY="my-minio-key:OSMM+vkKUTCvQs9YL/CVMIMt43HFhkUpqJxTmGl6rYw="

# Logging
export MINIO_LOGGER_WEBHOOK_ENABLE=on
export MINIO_LOGGER_WEBHOOK_ENDPOINT=http://localhost:8080/log
```

## MinIO Client (mc) Commands

### Configuration and Aliases

```bash
# Add MinIO server alias
mc alias set myminio http://localhost:9000 minioadmin minioadmin

# Add AWS S3 alias
mc alias set s3 https://s3.amazonaws.com ACCESS_KEY SECRET_KEY

# Add custom S3-compatible service
mc alias set custom https://storage.example.com ACCESS_KEY SECRET_KEY

# List aliases
mc alias list

# Remove alias
mc alias remove myminio

# Test connection
mc admin info myminio
```

### Bucket Operations

```bash
# List buckets
mc ls myminio
mc ls myminio/

# Create bucket
mc mb myminio/mybucket
mc mb myminio/photos

# Remove empty bucket
mc rb myminio/mybucket

# Remove bucket and all contents
mc rb --force myminio/mybucket

# Bucket info
mc stat myminio/mybucket

# List bucket contents
mc ls myminio/mybucket
mc ls --recursive myminio/mybucket
mc ls --recursive --versions myminio/mybucket

# Tree view
mc tree myminio/mybucket
```

### Object Operations

```bash
# Upload file
mc cp file.txt myminio/mybucket/
mc cp /path/to/file.txt myminio/mybucket/folder/

# Upload directory
mc cp --recursive /local/path/ myminio/mybucket/

# Download file
mc cp myminio/mybucket/file.txt ./
mc cp myminio/mybucket/file.txt /local/path/

# Download directory
mc cp --recursive myminio/mybucket/folder/ ./

# Sync directories
mc mirror /local/path/ myminio/mybucket/
mc mirror myminio/mybucket/ /local/path/
mc mirror --overwrite myminio/mybucket/ /local/path/
mc mirror --remove myminio/mybucket/ /local/path/

# Move objects
mc mv myminio/mybucket/old.txt myminio/mybucket/new.txt
mc mv myminio/bucket1/file.txt myminio/bucket2/

# Remove objects
mc rm myminio/mybucket/file.txt
mc rm --recursive myminio/mybucket/folder/
mc rm --force --recursive myminio/mybucket/

# Find objects
mc find myminio/mybucket --name "*.jpg"
mc find myminio/mybucket --larger-than 100MB
mc find myminio/mybucket --older-than 30d
mc find myminio/mybucket --newer-than 7d

# Get object info
mc stat myminio/mybucket/file.txt

# Share objects (presigned URLs)
mc share download myminio/mybucket/file.txt
mc share download --expire 24h myminio/mybucket/file.txt
mc share upload myminio/mybucket/
```

### Advanced Object Operations

```bash
# Multipart upload
mc cp --disable-multipart file.txt myminio/mybucket/
mc cp --part-size 64MB largefile.zip myminio/mybucket/

# Parallel uploads
mc cp --parallel 10 /path/to/files/ myminio/mybucket/

# Resume incomplete uploads
mc cp --continue file.txt myminio/mybucket/

# Copy with metadata
mc cp --attr "key1=value1,key2=value2" file.txt myminio/mybucket/

# Copy with server-side encryption
mc cp --encrypt-key "myminio/mybucket=32byteslongsecretkeymustbegiven" \
      file.txt myminio/mybucket/

# Versioned objects
mc ls --versions myminio/mybucket/file.txt
mc rm --version-id VERSION_ID myminio/mybucket/file.txt

# Retention and legal hold
mc retention set --default GOVERNANCE 30d myminio/mybucket
mc retention info myminio/mybucket/file.txt
mc legalhold set myminio/mybucket/file.txt
mc legalhold clear myminio/mybucket/file.txt
```

## Access Control and Security

### User Management

```bash
# List users
mc admin user list myminio

# Add user
mc admin user add myminio newuser newpassword

# Remove user
mc admin user remove myminio username

# Enable/disable user
mc admin user enable myminio username
mc admin user disable myminio username

# Get user info
mc admin user info myminio username

# Set user policy
mc admin policy set myminio readwrite user=username

# List user policies
mc admin user svcacct list myminio username
```

### Service Accounts

```bash
# Create service account
mc admin user svcacct add myminio username
mc admin user svcacct add --access-key myaccesskey --secret-key mysecretkey myminio username

# List service accounts
mc admin user svcacct list myminio username

# Get service account info
mc admin user svcacct info myminio ACCESS_KEY

# Remove service account
mc admin user svcacct remove myminio ACCESS_KEY

# Edit service account
mc admin user svcacct edit --secret-key newsecret myminio ACCESS_KEY
```

### Groups

```bash
# Create group
mc admin group add myminio developers user1 user2 user3

# List groups
mc admin group list myminio

# Remove group
mc admin group remove myminio developers

# Add users to group
mc admin group add myminio developers user4

# Remove users from group
mc admin group remove myminio developers user1

# Get group info
mc admin group info myminio developers
```

### Policies

```bash
# List policies
mc admin policy list myminio

# Create policy from file
cat > my-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::mybucket/*"
      ]
    }
  ]
}
EOF

mc admin policy add myminio my-policy my-policy.json

# Set policy for user
mc admin policy set myminio my-policy user=username

# Set policy for group
mc admin policy set myminio my-policy group=groupname

# Get policy info
mc admin policy info myminio my-policy

# Remove policy
mc admin policy remove myminio my-policy

# Built-in policies
mc admin policy set myminio readonly user=username
mc admin policy set myminio writeonly user=username
mc admin policy set myminio readwrite user=username
mc admin policy set myminio consoleAdmin user=username
mc admin policy set myminio diagnostics user=username
```

### Bucket Policies

```bash
# Set bucket policy
cat > bucket-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "*"},
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::mybucket/*"
    }
  ]
}
EOF

mc policy set-json bucket-policy.json myminio/mybucket

# Get bucket policy
mc policy get myminio/mybucket

# Remove bucket policy
mc policy set none myminio/mybucket

# Quick policy settings
mc policy set public myminio/mybucket
mc policy set download myminio/mybucket
mc policy set upload myminio/mybucket
mc policy set none myminio/mybucket
```

## Encryption and Security

### Server-Side Encryption (SSE)

```bash
# SSE-S3 (MinIO managed keys)
mc encrypt set SSE-S3 myminio/mybucket

# SSE-KMS (Key Management Service)
mc encrypt set SSE-KMS key-id myminio/mybucket

# SSE-C (Customer provided keys)
mc encrypt set SSE-C key-file myminio/mybucket

# Remove encryption
mc encrypt clear myminio/mybucket

# Get encryption info
mc encrypt info myminio/mybucket

# Upload with encryption
mc cp --encrypt-key "myminio/mybucket=32byteslongsecretkeymustbegiven" \
      file.txt myminio/mybucket/
```

### Key Management Server (KMS)

```bash
# Configure external KMS
export MINIO_KMS_SECRET_KEY="my-minio-key:OSMM+vkKUTCvQs9YL/CVMIMt43HFhkUpqJxTmGl6rYw="

# HashiCorp Vault integration
export MINIO_KMS_VAULT_ENDPOINT="https://vault.example.com"
export MINIO_KMS_VAULT_AUTH_TYPE="approle"
export MINIO_KMS_VAULT_APPROLE_ID="role-id"
export MINIO_KMS_VAULT_APPROLE_SECRET="secret-id"
export MINIO_KMS_VAULT_KEY_NAME="minio-key"

# AWS KMS integration
export MINIO_KMS_KMS_ENDPOINT="https://kms.us-east-1.amazonaws.com"
export MINIO_KMS_KMS_KEY_ID="arn:aws:kms:us-east-1:123456789:key/key-id"
```

### TLS/SSL Configuration

```bash
# Generate self-signed certificate
openssl req -new -x509 -days 3650 -nodes -out public.crt -keyout private.key \
  -subj "/CN=localhost"

# Place certificates
mkdir -p ~/.minio/certs
cp public.crt ~/.minio/certs/
cp private.key ~/.minio/certs/

# Start with TLS
minio server --certs-dir ~/.minio/certs /data

# Custom certificate directory
minio server --certs-dir /etc/minio/certs /data

# Multiple certificates (SNI)
# Place certificates in subdirectories:
# ~/.minio/certs/example.com/public.crt
# ~/.minio/certs/example.com/private.key
# ~/.minio/certs/storage.example.com/public.crt
# ~/.minio/certs/storage.example.com/private.key
```

## Monitoring and Administration

### Server Administration

```bash
# Server info
mc admin info myminio
mc admin info --json myminio

# Server configuration
mc admin config get myminio
mc admin config get myminio notify_webhook
mc admin config set myminio notify_webhook:1 queue_limit=10

# Restart server
mc admin service restart myminio

# Update server
mc admin update myminio

# Server logs
mc admin logs myminio
mc admin logs --last 100 myminio

# Console logs
mc admin console myminio

# Trace HTTP requests
mc admin trace myminio
mc admin trace --verbose myminio

# Profile server
mc admin profile start myminio
mc admin profile stop myminio
```

### Health and Diagnostics

```bash
# Health check
mc admin health myminio

# Performance test
mc admin speedtest myminio
mc admin speedtest --duration 30s myminio
mc admin speedtest --size 1GB myminio

# Drive scan
mc admin scanner start myminio
mc admin scanner status myminio

# Server metrics
mc admin prometheus metrics myminio
mc admin prometheus generate myminio

# Top operations
mc admin top locks myminio
mc admin top api myminio

# Bucket bandwidth
mc admin bandwidth set myminio --limit 100MiB/s
mc admin bandwidth info myminio
```

### Data Management

```bash
# Healing (repair)
mc admin heal myminio
mc admin heal --recursive myminio/mybucket
mc admin heal --dry-run myminio

# Decommission drives
mc admin decommission start myminio http://minio1/data1
mc admin decommission status myminio
mc admin decommission cancel myminio

# Pool management
mc admin pool list myminio
mc admin pool add myminio http://minio5/data1 http://minio6/data1

# Balance data
mc admin rebalance start myminio
mc admin rebalance status myminio
mc admin rebalance stop myminio
```

## Event Notifications

### Configure Notifications

```bash
# Webhook notification
mc admin config set myminio notify_webhook:1 \
  endpoint="http://localhost:8080/webhook" \
  queue_limit="10"

# AMQP notification
mc admin config set myminio notify_amqp:1 \
  url="amqp://localhost:5672" \
  exchange="minio" \
  exchange_type="fanout" \
  queue_limit="10"

# Redis notification
mc admin config set myminio notify_redis:1 \
  address="localhost:6379" \
  key="minio_events" \
  queue_limit="10"

# Kafka notification
mc admin config set myminio notify_kafka:1 \
  brokers="localhost:9092" \
  topic="minio_events" \
  queue_limit="10"

# NATS notification
mc admin config set myminio notify_nats:1 \
  address="localhost:4222" \
  subject="minio_events" \
  queue_limit="10"

# Restart to apply configuration
mc admin service restart myminio
```

### Set up Event Listeners

```bash
# Listen to bucket events
mc event add myminio/mybucket arn:minio:sqs::1:webhook --event put,delete

# Listen to specific prefix/suffix
mc event add myminio/mybucket arn:minio:sqs::1:webhook \
  --prefix photos/ --suffix .jpg --event put

# List event configurations
mc event list myminio/mybucket

# Remove event configuration
mc event remove myminio/mybucket arn:minio:sqs::1:webhook

# Event types:
# s3:ObjectCreated:Put
# s3:ObjectCreated:Post
# s3:ObjectCreated:Copy
# s3:ObjectCreated:CompleteMultipartUpload
# s3:ObjectRemoved:Delete
# s3:ObjectRemoved:DeleteMarkerCreated
# s3:ObjectAccessed:Get
# s3:ObjectAccessed:Head
# s3:ObjectTransition:Failed
# s3:ObjectTransition:Complete
# s3:BucketCreated
# s3:BucketRemoved
```

## Lifecycle Management

### Object Lifecycle

```bash
# Create lifecycle configuration
cat > lifecycle.json << EOF
{
  "Rules": [
    {
      "ID": "DeleteOldFiles",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "logs/"
      },
      "Expiration": {
        "Days": 90
      }
    },
    {
      "ID": "TransitionToIA",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "data/"
      },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
EOF

# Set lifecycle configuration
mc ilm import myminio/mybucket < lifecycle.json

# Export lifecycle configuration
mc ilm export myminio/mybucket

# Add lifecycle rule
mc ilm add --expiry-days 90 myminio/mybucket
mc ilm add --prefix "logs/" --expiry-days 30 myminio/mybucket
mc ilm add --prefix "data/" --transition-days 30 --storage-class STANDARD_IA myminio/mybucket

# List lifecycle rules
mc ilm ls myminio/mybucket

# Remove lifecycle rule
mc ilm remove --id "DeleteOldFiles" myminio/mybucket
```

### Versioning and Retention

```bash
# Enable versioning
mc version enable myminio/mybucket

# Suspend versioning
mc version suspend myminio/mybucket

# Get versioning status
mc version info myminio/mybucket

# Object Lock (WORM - Write Once Read Many)
mc retention set --default GOVERNANCE 90d myminio/mybucket
mc retention set --default COMPLIANCE 90d myminio/mybucket

# Get retention settings
mc retention info myminio/mybucket

# Set object retention
mc retention set GOVERNANCE 30d myminio/mybucket/file.txt

# Legal hold
mc legalhold set myminio/mybucket/file.txt
mc legalhold clear myminio/mybucket/file.txt
mc legalhold info myminio/mybucket/file.txt
```

## Replication

### Site Replication

```bash
# Add site for replication
mc admin replicate add myminio1 myminio2

# List replication sites
mc admin replicate info myminio1

# Remove site replication
mc admin replicate remove myminio1 --all

# Resync site replication
mc admin replicate resync start myminio1
mc admin replicate status myminio1
```

### Bucket Replication

```bash
# Create replication configuration
cat > replication.json << EOF
{
  "Role": "arn:aws:iam::account:role/replication-role",
  "Rules": [
    {
      "ID": "ReplicateAll",
      "Status": "Enabled",
      "Priority": 1,
      "Filter": {
        "Prefix": ""
      },
      "DeleteMarkerReplication": {
        "Status": "Enabled"
      },
      "Destination": {
        "Bucket": "arn:aws:s3:::destination-bucket",
        "StorageClass": "STANDARD"
      }
    }
  ]
}
EOF

# Set replication configuration
mc replicate add myminio/source-bucket \
  --remote-bucket "https://accesskey:secretkey@minio2:9000/destination-bucket"

# List replication configuration
mc replicate ls myminio/source-bucket

# Update replication
mc replicate edit --id "ReplicateAll" \
  --storage-class "REDUCED_REDUNDANCY" myminio/source-bucket

# Remove replication
mc replicate rm --id "ReplicateAll" myminio/source-bucket

# Replication status
mc replicate status myminio/source-bucket

# Resync replication
mc replicate resync start myminio/source-bucket
mc replicate resync status myminio/source-bucket
```

## Performance Tuning

### Server Optimization

```bash
# Increase file descriptor limits
ulimit -n 65536

# Memory settings
export MINIO_CACHE_DRIVES="/tmp/drive1,/tmp/drive2"
export MINIO_CACHE_QUOTA=80
export MINIO_CACHE_AFTER=3

# Compression settings
export MINIO_COMPRESS_ENABLE=on
export MINIO_COMPRESS_EXTENSIONS=".pdf,.doc,.txt,.log"
export MINIO_COMPRESS_MIME_TYPES="application/pdf,text/plain"

# Performance settings
export MINIO_API_REQUESTS_MAX=1600
export MINIO_API_CLUSTER_DEADLINE=10s
export MINIO_API_CORS_ALLOW_ORIGIN="*"

# Storage class settings
export MINIO_STORAGE_CLASS_STANDARD="EC:4"
export MINIO_STORAGE_CLASS_RRS="EC:2"

# Batch size optimization
mc mirror --parallel 10 /large-dataset/ myminio/mybucket/
```

### Network Optimization

```bash
# Load balancer configuration for distributed MinIO
# nginx.conf example
upstream minio {
    least_conn;
    server 192.168.1.11:9000;
    server 192.168.1.12:9000;
    server 192.168.1.13:9000;
    server 192.168.1.14:9000;
}

upstream console {
    least_conn;
    server 192.168.1.11:9001;
    server 192.168.1.12:9001;
    server 192.168.1.13:9001;
    server 192.168.1.14:9001;
}

server {
    listen 80;
    listen [::]:80;
    server_name storage.example.com;
    return 301 https://storage.example.com$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name storage.example.com;

    # SSL configuration
    ssl_certificate /path/to/public.crt;
    ssl_certificate_key /path/to/private.key;

    # Allow special characters in headers
    ignore_invalid_headers off;
    # Allow any size file to be uploaded
    client_max_body_size 0;
    # Disable buffering
    proxy_buffering off;
    proxy_request_buffering off;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 300;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        chunked_transfer_encoding off;

        proxy_pass http://minio;
    }
}
```

### Distributed Setup

```bash
# 4-node distributed setup
# Node 1:
minio server \
  http://minio1/data1 http://minio1/data2 \
  http://minio2/data1 http://minio2/data2 \
  http://minio3/data1 http://minio3/data2 \
  http://minio4/data1 http://minio4/data2

# Node 2:
minio server \
  http://minio1/data1 http://minio1/data2 \
  http://minio2/data1 http://minio2/data2 \
  http://minio3/data1 http://minio3/data2 \
  http://minio4/data1 http://minio4/data2

# 8-node setup with multiple drives per node
minio server \
  http://minio{1...8}/data{1...4}

# Multi-zone setup
minio server \
  http://zone1-minio{1...4}/data{1...4} \
  http://zone2-minio{1...4}/data{1...4}
```

## SDK and API Usage

### Python SDK Example

```python
from minio import Minio
from minio.error import S3Error

# Initialize MinIO client
client = Minio(
    "localhost:9000",
    access_key="minioadmin",
    secret_key="minioadmin",
    secure=False
)

# Create bucket
try:
    client.make_bucket("mybucket")
except S3Error as err:
    print(f"Error: {err}")

# Upload file
try:
    client.fput_object(
        "mybucket", "myfile.txt", "/path/to/local/file.txt"
    )
except S3Error as err:
    print(f"Error: {err}")

# Download file
try:
    client.fget_object(
        "mybucket", "myfile.txt", "/path/to/download/file.txt"
    )
except S3Error as err:
    print(f"Error: {err}")

# List objects
objects = client.list_objects("mybucket", recursive=True)
for obj in objects:
    print(obj.object_name, obj.size, obj.last_modified)

# Generate presigned URL
url = client.presigned_get_object("mybucket", "myfile.txt")
print(f"Download URL: {url}")

# Upload with metadata
client.put_object(
    "mybucket",
    "myfile.txt",
    data=open("/path/to/file.txt", "rb"),
    length=os.path.getsize("/path/to/file.txt"),
    metadata={"Content-Type": "text/plain", "X-Custom-Header": "value"}
)
```

### JavaScript SDK Example

```javascript
const Minio = require('minio');

// Initialize MinIO client
const minioClient = new Minio.Client({
    endPoint: 'localhost',
    port: 9000,
    useSSL: false,
    accessKey: 'minioadmin',
    secretKey: 'minioadmin'
});

// Create bucket
minioClient.makeBucket('mybucket', 'us-east-1', (err) => {
    if (err) return console.log(err);
    console.log('Bucket created successfully');
});

// Upload file
minioClient.fPutObject('mybucket', 'myfile.txt', '/path/to/file.txt', (err, etag) => {
    if (err) return console.log(err);
    console.log('File uploaded successfully. ETag:', etag);
});

// Download file
minioClient.fGetObject('mybucket', 'myfile.txt', '/path/to/download.txt', (err) => {
    if (err) return console.log(err);
    console.log('File downloaded successfully');
});

// List objects
const stream = minioClient.listObjects('mybucket', '', true);
stream.on('data', (obj) => {
    console.log(obj.name, obj.size, obj.lastModified);
});
stream.on('error', (err) => {
    console.log(err);
});

// Generate presigned URL
minioClient.presignedGetObject('mybucket', 'myfile.txt', 24*60*60, (err, presignedUrl) => {
    if (err) return console.log(err);
    console.log('Presigned URL:', presignedUrl);
});
```

### Go SDK Example

```go
package main

import (
    "context"
    "log"
    "github.com/minio/minio-go/v7"
    "github.com/minio/minio-go/v7/pkg/credentials"
)

func main() {
    // Initialize MinIO client
    minioClient, err := minio.New("localhost:9000", &minio.Options{
        Creds:  credentials.NewStaticV4("minioadmin", "minioadmin", ""),
        Secure: false,
    })
    if err != nil {
        log.Fatalln(err)
    }

    // Create bucket
    ctx := context.Background()
    bucketName := "mybucket"
    err = minioClient.MakeBucket(ctx, bucketName, minio.MakeBucketOptions{})
    if err != nil {
        log.Fatalln(err)
    }

    // Upload file
    objectName := "myfile.txt"
    filePath := "/path/to/file.txt"
    contentType := "text/plain"

    info, err := minioClient.FPutObject(ctx, bucketName, objectName, filePath, minio.PutObjectOptions{
        ContentType: contentType,
    })
    if err != nil {
        log.Fatalln(err)
    }
    log.Printf("Successfully uploaded %s of size %d\n", objectName, info.Size)

    // List objects
    for object := range minioClient.ListObjects(ctx, bucketName, minio.ListObjectsOptions{}) {
        if object.Err != nil {
            log.Fatalln(object.Err)
        }
        log.Println(object.Key, object.Size, object.LastModified)
    }
}
```

## Troubleshooting

### Common Issues

```bash
# Check server status
curl -I http://localhost:9000/minio/health/live

# Debug connection issues
mc admin info myminio
mc admin trace myminio

# Check disk usage
df -h /path/to/minio/data

# Check logs
mc admin logs myminio
journalctl -u minio.service -f

# Check configuration
mc admin config get myminio

# Network connectivity test
telnet minio-server 9000

# Check port availability
netstat -tlnp | grep 9000
ss -tlnp | grep 9000
```

### Performance Diagnostics

```bash
# Performance test
mc admin speedtest myminio
mc admin speedtest --duration 60s --size 1GB myminio

# I/O statistics
iostat -x 1
iotop

# Network statistics
iftop
netstat -i

# Memory usage
free -h
top -p $(pgrep minio)

# Check for drive errors
dmesg | grep -i error
sudo smartctl -a /dev/sdX
```

### Error Resolution

```bash
# "Disk not found" error
# Check if disks are mounted and accessible
ls -la /path/to/data/
mount | grep /path/to/data

# "Time skew" error
# Synchronize time across all nodes
sudo ntpdate -s time.nist.gov
sudo systemctl restart ntp

# "Connection refused" error
# Check firewall settings
sudo ufw status
sudo iptables -L

# "Access denied" error
# Check credentials and policies
mc admin user info myminio username
mc admin policy info myminio policyname

# "Insufficient storage" error
# Check disk space
df -h
mc admin info myminio

# Healing data
mc admin heal --recursive myminio
mc admin heal --dry-run myminio/bucket
```

### Backup and Recovery

```bash
# Backup configuration
mc admin config export myminio > minio-config.json

# Restore configuration
mc admin config import myminio < minio-config.json

# Backup metadata
mc mirror --preserve myminio/bucket /backup/location/

# Create snapshot (if using filesystem snapshots)
sudo lvcreate -L1G -s -n minio-snap /dev/vg/minio-lv

# Export IAM data
mc admin cluster iam export myminio iam-backup.zip

# Import IAM data
mc admin cluster iam import myminio iam-backup.zip

# Disaster recovery
# 1. Stop MinIO service
# 2. Restore data from backup
# 3. Start MinIO service
# 4. Verify data integrity
mc admin heal --recursive myminio
```

### Monitoring Setup

```bash
# Prometheus configuration
cat > prometheus.yml << EOF
scrape_configs:
  - job_name: 'minio'
    static_configs:
      - targets: ['localhost:9000']
    metrics_path: /minio/v2/metrics/cluster
    scheme: http
EOF

# Generate Prometheus config for MinIO
mc admin prometheus generate myminio

# Grafana dashboard
# Import dashboard ID: 13502 for MinIO monitoring

# Health check endpoint
curl http://localhost:9000/minio/health/live
curl http://localhost:9000/minio/health/ready

# Metrics endpoint
curl http://localhost:9000/minio/v2/metrics/cluster
```

---

*For more detailed information, visit the [official MinIO documentation](https://docs.min.io/)*
