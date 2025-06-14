# Docker Registry Cheatsheet

Docker Registry is a server-side application that stores and distributes Docker images. This cheatsheet covers Docker Hub, private registries, and registry management.

## Overview

### Registry Types
- **Docker Hub** - Official public registry
- **Private Registries** - Self-hosted or cloud-based
- **Cloud Registries** - AWS ECR, Google GCR, Azure ACR
- **Enterprise Registries** - Harbor, JFrog Artifactory, Sonatype Nexus

### Registry Components
- **Repository** - Collection of related images
- **Tag** - Version identifier for images
- **Layer** - Read-only file system changes
- **Manifest** - JSON document describing image configuration

## Docker Hub

### Basic Operations
```bash
# Login to Docker Hub
docker login
docker login --username=myuser

# Logout
docker logout

# Search for images
docker search nginx
docker search --limit 10 ubuntu
docker search --filter stars=50 mysql

# Pull images
docker pull nginx
docker pull nginx:1.20
docker pull ubuntu:20.04

# Push images
docker push myuser/myapp:latest
docker push myuser/myapp:v1.0

# Tag images for pushing
docker tag myapp:latest myuser/myapp:latest
docker tag myapp:latest myuser/myapp:v1.0
```

### Image Naming Convention
```bash
# Format: [registry-host[:port]/]username/repository[:tag]

# Docker Hub (default registry)
myuser/myapp:latest
myuser/myapp:v1.0
ubuntu:20.04
nginx:alpine

# With explicit registry
registry.example.com:5000/myuser/myapp:latest
localhost:5000/myapp:dev
```

### Multi-platform Images
```bash
# Build and push multi-platform images
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t myuser/myapp:latest --push .

# Inspect multi-platform manifest
docker buildx imagetools inspect myuser/myapp:latest
```

## Private Registry Setup

### Run Local Registry
```bash
# Basic registry
docker run -d -p 5000:5000 --name registry registry:2

# Registry with volume
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v /opt/registry:/var/lib/registry \
  registry:2

# Registry with restart policy
docker run -d \
  -p 5000:5000 \
  --name registry \
  --restart=always \
  -v /opt/registry:/var/lib/registry \
  registry:2
```

### Registry with TLS
```bash
# Generate certificates
mkdir -p certs
openssl req -newkey rsa:4096 -nodes -sha256 \
  -keyout certs/domain.key -x509 -days 365 \
  -out certs/domain.crt

# Run registry with TLS
docker run -d \
  --name registry \
  --restart=always \
  -p 443:5000 \
  -v $(pwd)/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```

### Registry with Authentication
```bash
# Create htpasswd file
mkdir auth
docker run --entrypoint htpasswd httpd:2 -Bbn testuser testpassword > auth/htpasswd

# Run registry with basic auth
docker run -d \
  --name registry \
  --restart=always \
  -p 5000:5000 \
  -v $(pwd)/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2

# Login to authenticated registry
docker login localhost:5000
```

## Registry Configuration

### Configuration File (config.yml)
```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

### Storage Backends
```yaml
# Filesystem storage
storage:
  filesystem:
    rootdirectory: /var/lib/registry

# AWS S3 storage
storage:
  s3:
    accesskey: myaccesskey
    secretkey: mysecretkey
    region: us-east-1
    bucket: mybucket
    rootdirectory: /registry

# Google Cloud Storage
storage:
  gcs:
    bucket: mybucket
    keyfile: /path/to/keyfile.json
    rootdirectory: /registry

# Azure Blob Storage
storage:
  azure:
    accountname: myaccount
    accountkey: mykey
    container: mycontainer
    rootdirectory: /registry
```

### Advanced Configuration
```yaml
version: 0.1
log:
  level: info
  formatter: text
  fields:
    service: registry
    environment: production

storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
  delete:
    enabled: true

http:
  addr: :5000
  host: https://myregistry.example.com
  headers:
    X-Content-Type-Options: [nosniff]
    Access-Control-Allow-Origin: ['*']
  tls:
    certificate: /certs/domain.crt
    key: /certs/domain.key

auth:
  htpasswd:
    realm: basic-realm
    path: /auth/htpasswd

middleware:
  registry:
    - name: cloudfront
      options:
        baseurl: https://d1234567890.cloudfront.net
        privatekey: /etc/docker/cloudfront.pem
        keypairid: APKAJ...

notifications:
  endpoints:
    - name: webhook
      url: https://example.com/webhook
      headers:
        Authorization: [Bearer token]
      events:
        - name: push
        - name: pull
```

## Registry API

### API Endpoints
```bash
# Check registry version and features
curl http://localhost:5000/v2/

# List repositories
curl http://localhost:5000/v2/_catalog

# List tags for a repository
curl http://localhost:5000/v2/myapp/tags/list

# Get manifest
curl -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
     http://localhost:5000/v2/myapp/manifests/latest

# Get image config
curl http://localhost:5000/v2/myapp/blobs/sha256:...

# Delete manifest (if delete enabled)
curl -X DELETE http://localhost:5000/v2/myapp/manifests/sha256:...
```

### Registry API with Authentication
```bash
# Get auth token
TOKEN=$(curl -s -X GET \
  -H "Content-Type: application/json" \
  "https://auth.docker.io/token?service=registry.docker.io&scope=repository:library/ubuntu:pull" \
  | jq -r .token)

# Use token in API calls
curl -H "Authorization: Bearer $TOKEN" \
     https://registry-1.docker.io/v2/library/ubuntu/tags/list
```

## Working with Images

### Push/Pull Operations
```bash
# Tag image for registry
docker tag myapp:latest localhost:5000/myapp:latest
docker tag myapp:latest localhost:5000/myapp:v1.0

# Push to registry
docker push localhost:5000/myapp:latest
docker push localhost:5000/myapp:v1.0

# Pull from registry
docker pull localhost:5000/myapp:latest
docker pull localhost:5000/myapp:v1.0

# Push all tags
docker push localhost:5000/myapp --all-tags
```

### Image Inspection
```bash
# Inspect image
docker inspect localhost:5000/myapp:latest

# View image history
docker history localhost:5000/myapp:latest

# Get image digest
docker inspect --format='{{index .RepoDigests 0}}' localhost:5000/myapp:latest

# Pull by digest
docker pull localhost:5000/myapp@sha256:...
```

### Image Management
```bash
# Remove image
docker rmi localhost:5000/myapp:latest

# Remove all local images from registry
docker images localhost:5000/* -q | xargs docker rmi

# Prune unused images
docker image prune
docker image prune -a
```

## Registry Management

### Garbage Collection
```bash
# Run garbage collection
docker exec registry bin/registry garbage-collect /etc/docker/registry/config.yml

# Dry run garbage collection
docker exec registry bin/registry garbage-collect --dry-run /etc/docker/registry/config.yml

# Delete untagged manifests
docker exec registry bin/registry garbage-collect --delete-untagged /etc/docker/registry/config.yml
```

### Registry Maintenance
```bash
# View registry logs
docker logs registry
docker logs -f registry

# Check registry health
curl http://localhost:5000/v2/

# Monitor registry metrics
curl http://localhost:5000/metrics

# Backup registry data
tar -czf registry-backup.tar.gz /opt/registry/

# Restore registry data
tar -xzf registry-backup.tar.gz -C /
```

### Storage Management
```bash
# Check storage usage
du -sh /opt/registry/

# Find large blobs
find /opt/registry/docker/registry/v2/blobs -type f -size +100M

# Clean up temporary files
find /opt/registry/docker/registry/v2/repositories -name "_uploads" -type d -exec rm -rf {} +
```

## Docker Compose Setup

### Basic Registry
```yaml
# docker-compose.yml
version: '3.8'

services:
  registry:
    image: registry:2
    container_name: registry
    restart: always
    ports:
      - "5000:5000"
    volumes:
      - registry-data:/var/lib/registry
    environment:
      - REGISTRY_STORAGE_DELETE_ENABLED=true

volumes:
  registry-data:
```

### Registry with UI
```yaml
version: '3.8'

services:
  registry:
    image: registry:2
    container_name: registry
    restart: always
    ports:
      - "5000:5000"
    volumes:
      - registry-data:/var/lib/registry
      - ./config.yml:/etc/docker/registry/config.yml
    environment:
      - REGISTRY_STORAGE_DELETE_ENABLED=true

  registry-ui:
    image: joxit/docker-registry-ui:main
    container_name: registry-ui
    restart: always
    ports:
      - "8080:80"
    environment:
      - SINGLE_REGISTRY=true
      - REGISTRY_TITLE=Docker Registry UI
      - DELETE_IMAGES=true
      - SHOW_CONTENT_DIGEST=true
      - NGINX_PROXY_PASS_URL=http://registry:5000
      - SHOW_CATALOG_NB_TAGS=true
      - CATALOG_MIN_BRANCHES=1
      - CATALOG_MAX_BRANCHES=1
      - TAGLIST_PAGE_SIZE=100
      - REGISTRY_SECURED=false
      - CATALOG_ELEMENTS_LIMIT=1000
    depends_on:
      - registry

volumes:
  registry-data:
```

### Secure Registry with Authentication
```yaml
version: '3.8'

services:
  registry:
    image: registry:2
    container_name: registry
    restart: always
    ports:
      - "443:5000"
    volumes:
      - registry-data:/var/lib/registry
      - ./certs:/certs
      - ./auth:/auth
    environment:
      - REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt
      - REGISTRY_HTTP_TLS_KEY=/certs/domain.key
      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
      - REGISTRY_STORAGE_DELETE_ENABLED=true

volumes:
  registry-data:
```

## Cloud Registries

### AWS ECR
```bash
# Install AWS CLI
pip install awscli

# Configure AWS credentials
aws configure

# Get login token
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Create repository
aws ecr create-repository --repository-name myapp

# Tag and push
docker tag myapp:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Pull image
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# List repositories
aws ecr describe-repositories

# List images in repository
aws ecr list-images --repository-name myapp
```

### Google Container Registry (GCR)
```bash
# Install gcloud CLI
curl https://sdk.cloud.google.com | bash

# Authenticate
gcloud auth configure-docker

# Tag and push
docker tag myapp:latest gcr.io/project-id/myapp:latest
docker push gcr.io/project-id/myapp:latest

# Pull image
docker pull gcr.io/project-id/myapp:latest

# List images
gcloud container images list

# List tags
gcloud container images list-tags gcr.io/project-id/myapp
```

### Azure Container Registry (ACR)
```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login

# Create registry
az acr create --resource-group myResourceGroup --name myregistry --sku Basic

# Login to ACR
az acr login --name myregistry

# Tag and push
docker tag myapp:latest myregistry.azurecr.io/myapp:latest
docker push myregistry.azurecr.io/myapp:latest

# Pull image
docker pull myregistry.azurecr.io/myapp:latest

# List repositories
az acr repository list --name myregistry

# List tags
az acr repository show-tags --name myregistry --repository myapp
```

## Security

### Image Scanning
```bash
# Docker Hub security scanning
docker scan myuser/myapp:latest

# Trivy security scanner
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image myapp:latest

# Clair security scanner
docker run -d --name clair-db arminc/clair-db:latest
docker run -p 6060:6060 --link clair-db:postgres -d --name clair arminc/clair-local-scan:latest
```

### Content Trust
```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Generate delegation key
docker trust key generate mykey

# Add signer to repository
docker trust signer add --key mykey.pub myuser myuser/myapp

# Sign and push image
docker trust sign myuser/myapp:latest

# Inspect trust data
docker trust inspect myuser/myapp:latest

# Revoke trust
docker trust revoke myuser/myapp:latest
```

### Registry Security
```bash
# Generate strong htpasswd
docker run --rm --entrypoint htpasswd httpd:2 -Bbn username $(openssl rand -base64 32)

# Use TLS certificates
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# Restrict access with firewall
sudo ufw allow from 192.168.1.0/24 to any port 5000

# Use reverse proxy with authentication
nginx -s reload
```

## Monitoring and Logging

### Prometheus Metrics
```yaml
# Registry configuration with metrics
http:
  debug:
    addr: :5001
    prometheus:
      enabled: true
      path: /metrics
```

### Log Configuration
```yaml
log:
  level: info
  formatter: json
  fields:
    service: registry
    environment: production
  hooks:
    - type: mail
      levels: [panic, fatal, error]
      options:
        smtp:
          addr: smtp.example.com:587
        to: [admin@example.com]
        from: registry@example.com
        subject: "Registry Alert"
```

### Health Checks
```bash
# Registry health endpoint
curl -f http://localhost:5000/v2/ || exit 1

# Docker health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:5000/v2/ || exit 1

# Custom health check script
#!/bin/bash
response=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:5000/v2/)
if [ $response = "200" ]; then
  echo "Registry is healthy"
  exit 0
else
  echo "Registry is unhealthy"
  exit 1
fi
```

## Troubleshooting

### Common Issues
```bash
# Registry connection issues
docker info | grep Registry
telnet localhost 5000

# Certificate issues
openssl s_client -connect registry.example.com:443

# Authentication issues
docker login --username testuser localhost:5000

# Push/pull issues
docker push -a localhost:5000/myapp  # Push all tags
docker pull --all-tags localhost:5000/myapp  # Pull all tags

# Storage issues
df -h /opt/registry
lsof | grep registry

# Registry not responding
docker logs registry
docker restart registry
```

### Debug Commands
```bash
# Enable debug logging
export DOCKER_BUILDKIT_DEBUG=1
docker --debug pull myapp:latest

# Registry debug mode
docker run -e REGISTRY_LOG_LEVEL=debug registry:2

# Network troubleshooting
docker network ls
docker network inspect bridge
nslookup registry.example.com

# Check registry configuration
docker exec registry cat /etc/docker/registry/config.yml

# Validate manifest
curl -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
     http://localhost:5000/v2/myapp/manifests/latest | jq .
```

## Registry Utilities

### Registry Tools
```bash
# reg - registry CLI tool
go install github.com/genuinetools/reg@latest
reg ls localhost:5000
reg tags localhost:5000/myapp
reg rm localhost:5000/myapp:old

# crane - container registry tool
go install github.com/google/go-containerregistry/cmd/crane@latest
crane ls localhost:5000
crane manifest localhost:5000/myapp:latest
crane copy localhost:5000/myapp:latest registry.example.com/myapp:latest

# skopeo - image operations
sudo apt-get install skopeo
skopeo list-tags docker://localhost:5000/myapp
skopeo copy docker://localhost:5000/myapp:latest docker://registry.example.com/myapp:latest
```

### Backup and Migration
```bash
# Backup registry images
mkdir backup
docker images --format "table {{.Repository}}:{{.Tag}}" | tail -n +2 > backup/images.txt
while read image; do
  docker save "$image" | gzip > "backup/$(echo $image | tr '/' '_' | tr ':' '_').tar.gz"
done < backup/images.txt

# Restore registry images
for file in backup/*.tar.gz; do
  docker load < "$file"
done

# Migrate between registries
docker images localhost:5000/* --format "{{.Repository}}:{{.Tag}}" | while read image; do
  new_image=$(echo $image | sed 's/localhost:5000/registry.example.com/')
  docker tag "$image" "$new_image"
  docker push "$new_image"
done
```

## Performance Tuning

### Registry Optimization
```yaml
# config.yml optimizations
storage:
  cache:
    blobdescriptor: redis
  redis:
    addr: redis:6379
    password: secret
    db: 0
    dialtimeout: 10ms
    readtimeout: 10ms
    writetimeout: 10ms
    pool:
      maxidle: 16
      maxactive: 64
      idletimeout: 300s

http:
  addr: :5000
  relativeurls: false
  draintimeout: 60s
  http2:
    disabled: false
```

### Caching Strategies
```bash
# Redis cache for registry
docker run -d --name redis redis:alpine

# CDN configuration
# Use CloudFront, CloudFlare, or similar for layer caching

# Registry mirror
docker run -d -p 5000:5000 \
  -e REGISTRY_PROXY_REMOTEURL=https://registry-1.docker.io \
  registry:2
```

## Useful Commands Reference

| Command | Description |
|---------|-------------|
| `docker login` | Login to registry |
| `docker logout` | Logout from registry |
| `docker push` | Push image to registry |
| `docker pull` | Pull image from registry |
| `docker search` | Search for images |
| `docker tag` | Tag image for registry |
| `curl /v2/` | Check registry API |
| `curl /v2/_catalog` | List repositories |
| `curl /v2/repo/tags/list` | List tags |
| `docker exec registry gc` | Run garbage collection |

## Environment Variables

```bash
# Registry configuration
export REGISTRY_STORAGE_DELETE_ENABLED=true
export REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt
export REGISTRY_HTTP_TLS_KEY=/certs/domain.key
export REGISTRY_AUTH=htpasswd
export REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
export REGISTRY_LOG_LEVEL=info

# Docker client configuration
export DOCKER_CONTENT_TRUST=1
export DOCKER_BUILDKIT=1
export DOCKER_REGISTRY=localhost:5000
```

---

*For more detailed information, visit the [official Docker Registry documentation](https://docs.docker.com/registry/)*

