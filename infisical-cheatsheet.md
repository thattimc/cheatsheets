# Infisical Cheatsheet

Infisical is an open-source secrets management platform that helps teams securely store, sync, and manage environment variables and secrets across applications and infrastructure.

## Overview

### Key Features
- **Secret Management** - Store and manage environment variables securely
- **Multi-Environment** - Separate secrets for dev, staging, production
- **Team Collaboration** - Role-based access control and permissions
- **Git Integration** - Version control for secrets
- **Auto Sync** - Real-time secret synchronization
- **Audit Logs** - Track all secret access and modifications
- **Integrations** - Support for various platforms and CI/CD tools

### Components
- **Infisical Core** - Backend API and database
- **Web Dashboard** - Browser-based management interface
- **CLI Tool** - Command-line interface for developers
- **SDKs** - Client libraries for various languages
- **Kubernetes Operator** - Native Kubernetes integration

## Installation

### Self-hosted with Docker Compose
```bash
# Clone the repository
git clone https://github.com/Infisical/infisical.git
cd infisical

# Copy environment file
cp .env.example .env

# Edit configuration
vim .env

# Start services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f
```

### Docker Compose Configuration
```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: infisical
      POSTGRES_USER: infisical
      POSTGRES_PASSWORD: your_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - infisical

  redis:
    image: redis:7-alpine
    networks:
      - infisical

  backend:
    image: infisical/infisical:latest
    depends_on:
      - db
      - redis
    environment:
      - NODE_ENV=production
      - DB_CONNECTION_URI=postgres://infisical:your_password@db:5432/infisical
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=your_jwt_secret
      - ENCRYPTION_KEY=your_encryption_key
      - TELEMETRY_ENABLED=false
    ports:
      - "8080:8080"
    networks:
      - infisical

  frontend:
    image: infisical/frontend:latest
    depends_on:
      - backend
    environment:
      - NEXT_PUBLIC_INFISICAL_URL=http://localhost:8080
    ports:
      - "3000:3000"
    networks:
      - infisical

volumes:
  postgres_data:

networks:
  infisical:
```

### Kubernetes Deployment
```yaml
# infisical-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: infisical
---
# infisical-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: infisical-config
  namespace: infisical
data:
  NODE_ENV: "production"
  TELEMETRY_ENABLED: "false"
  DB_CONNECTION_URI: "postgres://infisical:password@postgres:5432/infisical"
  REDIS_URL: "redis://redis:6379"
---
# infisical-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: infisical-secrets
  namespace: infisical
type: Opaque
data:
  JWT_SECRET: <base64-encoded-jwt-secret>
  ENCRYPTION_KEY: <base64-encoded-encryption-key>
---
# infisical-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: infisical-backend
  namespace: infisical
spec:
  replicas: 2
  selector:
    matchLabels:
      app: infisical-backend
  template:
    metadata:
      labels:
        app: infisical-backend
    spec:
      containers:
      - name: infisical
        image: infisical/infisical:latest
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: infisical-config
        - secretRef:
            name: infisical-secrets
        livenessProbe:
          httpGet:
            path: /api/status
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/status
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
---
# infisical-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: infisical-service
  namespace: infisical
spec:
  selector:
    app: infisical-backend
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  type: ClusterIP
---
# infisical-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: infisical-ingress
  namespace: infisical
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: infisical.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: infisical-service
            port:
              number: 8080
```

## CLI Installation and Setup

### Install CLI
```bash
# macOS
brew install infisical/get-cli/infisical

# Linux
curl -1sLf 'https://dl.cloudsmith.io/public/infisical/infisical-cli/setup.deb.sh' | sudo -E bash
sudo apt-get update && sudo apt-get install infisical

# Windows (using Scoop)
scoop bucket add infisical https://github.com/Infisical/scoop-infisical.git
scoop install infisical

# Using npm
npm install -g @infisical/cli

# Verify installation
infisical --version
```

### CLI Authentication
```bash
# Login to Infisical
infisical login

# Login with specific instance
infisical login --domain=https://infisical.example.com

# Login with service token
infisical login --token=<service-token>

# Check login status
infisical user

# Logout
infisical logout
```

### Project Setup
```bash
# List projects
infisical projects

# Initialize project in current directory
infisical init

# Initialize with specific project
infisical init --project-id=<project-id>

# View current project
infisical project current
```

## Secret Management

### Managing Secrets via CLI
```bash
# List secrets
infisical secrets

# List secrets for specific environment
infisical secrets --env=production

# Get specific secret
infisical secrets get <secret-name>
infisical secrets get <secret-name> --env=staging

# Set secret
infisical secrets set <secret-name> <secret-value>
infisical secrets set DATABASE_URL "postgres://user:pass@host:5432/db" --env=production

# Set multiple secrets from file
infisical secrets set --from-file=.env.production --env=production

# Delete secret
infisical secrets delete <secret-name>
infisical secrets delete <secret-name> --env=development

# Export secrets
infisical export --env=production
infisical export --format=json --env=production
infisical export --format=yaml --env=production
```

### Running Applications with Secrets
```bash
# Run command with injected secrets
infisical run -- npm start
infisical run --env=production -- python app.py

# Run with specific project
infisical run --project-id=<project-id> --env=staging -- ./start.sh

# Run with expanded variables
infisical run --expand -- echo $DATABASE_URL

# Run and override specific variables
infisical run --env=production DEBUG=true -- node server.js
```

### Environment File Generation
```bash
# Generate .env file
infisical export > .env
infisical export --env=production > .env.production

# Generate different formats
infisical export --format=json > secrets.json
infisical export --format=yaml > secrets.yaml
infisical export --format=csv > secrets.csv

# Generate with custom template
infisical export --template="{{.Key}}={{.Value}}" > custom.env
```

## Web Dashboard

### Project Management
```bash
# Create new project via API
curl -X POST https://app.infisical.com/api/v1/projects \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Project",
    "description": "Project description"
  }'

# List projects
curl -X GET https://app.infisical.com/api/v1/projects \
  -H "Authorization: Bearer <token>"

# Get project details
curl -X GET https://app.infisical.com/api/v1/projects/<project-id> \
  -H "Authorization: Bearer <token>"
```

### Environment Management
```bash
# Create environment
curl -X POST https://app.infisical.com/api/v1/projects/<project-id>/environments \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "staging",
    "slug": "staging"
  }'

# List environments
curl -X GET https://app.infisical.com/api/v1/projects/<project-id>/environments \
  -H "Authorization: Bearer <token>"
```

## API Usage

### Authentication
```bash
# Get service token from dashboard
# Use token in API requests
TOKEN="your-service-token"
PROJECT_ID="your-project-id"
ENVIRONMENT="production"
```

### Secret Operations via API
```bash
# Get secrets
curl -X GET "https://app.infisical.com/api/v3/secrets/raw?environment=${ENVIRONMENT}&workspaceId=${PROJECT_ID}" \
  -H "Authorization: Bearer ${TOKEN}"

# Create secret
curl -X POST "https://app.infisical.com/api/v3/secrets/raw" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "workspaceId": "'${PROJECT_ID}'",
    "environment": "'${ENVIRONMENT}'",
    "type": "shared",
    "secretKey": "API_KEY",
    "secretValue": "secret-value",
    "secretComment": "API key for external service"
  }'

# Update secret
curl -X PATCH "https://app.infisical.com/api/v3/secrets/raw" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "workspaceId": "'${PROJECT_ID}'",
    "environment": "'${ENVIRONMENT}'",
    "type": "shared",
    "secretKey": "API_KEY",
    "secretValue": "new-secret-value"
  }'

# Delete secret
curl -X DELETE "https://app.infisical.com/api/v3/secrets/raw" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "workspaceId": "'${PROJECT_ID}'",
    "environment": "'${ENVIRONMENT}'",
    "type": "shared",
    "secretKey": "API_KEY"
  }'
```

## SDKs and Integrations

### Node.js SDK
```javascript
// Installation
// npm install @infisical/sdk

const { InfisicalSDK } = require('@infisical/sdk');

// Initialize client
const infisical = new InfisicalSDK({
  siteUrl: 'https://app.infisical.com',
  auth: {
    accessToken: process.env.INFISICAL_TOKEN
  }
});

// Get secret
async function getSecret() {
  try {
    const secret = await infisical.getSecret({
      secretName: 'DATABASE_URL',
      projectId: 'project-id',
      environment: 'production'
    });
    console.log(secret.secretValue);
  } catch (error) {
    console.error('Error fetching secret:', error);
  }
}

// Get all secrets
async function getAllSecrets() {
  try {
    const secrets = await infisical.listSecrets({
      projectId: 'project-id',
      environment: 'production'
    });
    return secrets;
  } catch (error) {
    console.error('Error fetching secrets:', error);
  }
}

// Create secret
async function createSecret() {
  try {
    await infisical.createSecret({
      projectId: 'project-id',
      environment: 'production',
      secretName: 'NEW_SECRET',
      secretValue: 'secret-value',
      secretComment: 'Description of the secret'
    });
  } catch (error) {
    console.error('Error creating secret:', error);
  }
}
```

### Python SDK
```python
# Installation
# pip install infisical

from infisical import InfisicalClient
import os

# Initialize client
client = InfisicalClient(
    site_url="https://app.infisical.com",
    auth={
        "access_token": os.getenv("INFISICAL_TOKEN")
    }
)

# Get secret
def get_secret():
    try:
        secret = client.get_secret(
            secret_name="DATABASE_URL",
            project_id="project-id",
            environment="production"
        )
        return secret.secret_value
    except Exception as e:
        print(f"Error fetching secret: {e}")

# Get all secrets
def get_all_secrets():
    try:
        secrets = client.list_secrets(
            project_id="project-id",
            environment="production"
        )
        return {secret.secret_name: secret.secret_value for secret in secrets}
    except Exception as e:
        print(f"Error fetching secrets: {e}")

# Create secret
def create_secret():
    try:
        client.create_secret(
            project_id="project-id",
            environment="production",
            secret_name="NEW_SECRET",
            secret_value="secret-value",
            secret_comment="Description of the secret"
        )
    except Exception as e:
        print(f"Error creating secret: {e}")
```

### Go SDK
```go
// go mod init myapp
// go get github.com/infisical/go-sdk

package main

import (
    "fmt"
    "log"
    "os"
    
    infisical "github.com/infisical/go-sdk"
)

func main() {
    // Initialize client
    client := infisical.NewInfisicalClient(infisical.Config{
        SiteUrl: "https://app.infisical.com",
        Auth: infisical.AuthenticationDetails{
            AccessToken: os.Getenv("INFISICAL_TOKEN"),
        },
    })

    // Get secret
    secret, err := client.GetSecret(infisical.GetSecretOptions{
        SecretName:  "DATABASE_URL",
        ProjectID:   "project-id",
        Environment: "production",
    })
    if err != nil {
        log.Fatalf("Error fetching secret: %v", err)
    }
    fmt.Println(secret.SecretValue)

    // Get all secrets
    secrets, err := client.ListSecrets(infisical.ListSecretsOptions{
        ProjectID:   "project-id",
        Environment: "production",
    })
    if err != nil {
        log.Fatalf("Error fetching secrets: %v", err)
    }
    
    for _, secret := range secrets {
        fmt.Printf("%s: %s\n", secret.SecretName, secret.SecretValue)
    }
}
```

## Kubernetes Integration

### Infisical Kubernetes Operator
```bash
# Install operator
kubectl apply -f https://raw.githubusercontent.com/Infisical/infisical/main/k8s-operator/kubectl-apply/install-operator.yaml

# Verify installation
kubectl get pods -n infisical-operator-system
```

### InfisicalSecret Custom Resource
```yaml
# infisical-secret.yaml
apiVersion: secrets.infisical.com/v1alpha1
kind: InfisicalSecret
metadata:
  name: infisical-secret
  namespace: default
spec:
  # Infisical project configuration
  hostAPI: https://app.infisical.com/api
  projectId: "your-project-id"
  environment: "production"
  secretType: "shared"
  
  # Authentication
  tokenSecretReference:
    secretName: service-token
    secretNamespace: default
    tokenKey: token
  
  # Managed secret configuration
  managedSecretReference:
    secretName: managed-secret
    secretNamespace: default
    creationPolicy: "Owner"
  
  # Secret path (optional)
  secretPath: "/"
---
# service-token-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: service-token
  namespace: default
type: Opaque
data:
  token: <base64-encoded-service-token>
```

### Using Secrets in Pods
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:latest
        envFrom:
        - secretRef:
            name: managed-secret  # Created by InfisicalSecret
        # Or use specific environment variables
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: managed-secret
              key: DATABASE_URL
```

## CI/CD Integration

### GitHub Actions
```yaml
# .github/workflows/deploy.yml
name: Deploy Application

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Install Infisical CLI
      run: |
        curl -1sLf 'https://dl.cloudsmith.io/public/infisical/infisical-cli/setup.deb.sh' | sudo -E bash
        sudo apt-get update && sudo apt-get install infisical
    
    - name: Authenticate with Infisical
      run: infisical login --token=${{ secrets.INFISICAL_TOKEN }}
    
    - name: Run deployment with secrets
      run: |
        infisical run --env=production -- ./deploy.sh
      env:
        PROJECT_ID: ${{ secrets.INFISICAL_PROJECT_ID }}
```

### GitLab CI
```yaml
# .gitlab-ci.yml
stages:
  - deploy

deploy:
  stage: deploy
  image: ubuntu:latest
  before_script:
    - apt-get update && apt-get install -y curl
    - curl -1sLf 'https://dl.cloudsmith.io/public/infisical/infisical-cli/setup.deb.sh' | bash
    - apt-get update && apt-get install infisical
    - infisical login --token=$INFISICAL_TOKEN
  script:
    - infisical run --env=production --project-id=$PROJECT_ID -- ./deploy.sh
  only:
    - main
```

### Docker Integration
```dockerfile
# Dockerfile
FROM node:18-alpine

# Install Infisical CLI
RUN apk add --no-cache curl bash
RUN curl -1sLf 'https://dl.cloudsmith.io/public/infisical/infisical-cli/setup.alpine.sh' | bash
RUN apk add infisical

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .

# Run application with Infisical
CMD ["infisical", "run", "--", "npm", "start"]
```

## Security and Access Control

### Service Tokens
```bash
# Create service token via CLI
infisical service-token create \
  --name="CI/CD Token" \
  --environment=production \
  --project-id=<project-id> \
  --ttl=30d

# List service tokens
infisical service-token list

# Revoke service token
infisical service-token revoke <token-id>
```

### Role-Based Access Control
```bash
# Invite user to project
infisical users invite \
  --email=user@example.com \
  --role=developer \
  --project-id=<project-id>

# List project members
infisical users list --project-id=<project-id>

# Update user role
infisical users update-role \
  --user-id=<user-id> \
  --role=admin \
  --project-id=<project-id>

# Remove user from project
infisical users remove \
  --user-id=<user-id> \
  --project-id=<project-id>
```

### Audit Logs
```bash
# View audit logs
infisical audit-logs list --project-id=<project-id>

# Filter audit logs
infisical audit-logs list \
  --project-id=<project-id> \
  --start-date=2024-01-01 \
  --end-date=2024-01-31 \
  --event-type=secret.read

# Export audit logs
infisical audit-logs export \
  --project-id=<project-id> \
  --format=csv \
  --output=audit_logs.csv
```

## Backup and Migration

### Export Secrets
```bash
# Export all environments
for env in development staging production; do
  infisical export --env=$env --format=json > secrets_$env.json
done

# Export with encryption
infisical export --env=production --format=json | gpg -c > secrets_production.json.gpg

# Backup project configuration
infisical project export --project-id=<project-id> > project_config.json
```

### Import Secrets
```bash
# Import from .env file
infisical secrets set --from-file=.env.production --env=production

# Import from JSON
cat secrets_production.json | jq -r 'to_entries[] | "\(.key)=\(.value)"' | while read line; do
  key=$(echo $line | cut -d'=' -f1)
  value=$(echo $line | cut -d'=' -f2-)
  infisical secrets set "$key" "$value" --env=production
done

# Bulk import via API
curl -X POST "https://app.infisical.com/api/v3/secrets/batch" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d @secrets_bulk.json
```

## Monitoring and Troubleshooting

### Health Checks
```bash
# Check Infisical service health
curl -f http://localhost:8080/api/status

# Check database connectivity
docker-compose exec backend npm run db:health

# Check Redis connectivity
docker-compose exec backend npm run redis:health

# Monitor logs
docker-compose logs -f backend
docker-compose logs -f frontend
```

### Common Issues
```bash
# CLI authentication issues
infisical logout
infisical login --domain=https://your-instance.com

# Project sync issues
rm -rf .infisical.json
infisical init --project-id=<project-id>

# Permission issues
infisical user  # Check current user and permissions

# Network connectivity
curl -v https://app.infisical.com/api/status

# Token validation
curl -H "Authorization: Bearer $INFISICAL_TOKEN" \
     https://app.infisical.com/api/v1/auth/token/validate
```

### Debug Mode
```bash
# Enable debug logging
export INFISICAL_DEBUG=true
export INFISICAL_LOG_LEVEL=debug

# Run CLI with verbose output
infisical secrets --debug

# Check configuration
infisical config
```

## Configuration Files

### CLI Configuration
```json
// ~/.config/infisical/config.json
{
  "vault": {
    "backend": "file",
    "file": {
      "path": "~/.config/infisical/vault.json"
    }
  },
  "logLevel": "info",
  "defaultDomain": "https://app.infisical.com"
}
```

### Project Configuration
```json
// .infisical.json
{
  "workspaceId": "project-id",
  "defaultEnvironment": "development",
  "gitBranchToEnvironmentMapping": {
    "main": "production",
    "staging": "staging",
    "develop": "development"
  }
}
```

## Useful Commands Reference

| Command | Description |
|---------|-------------|
| `infisical login` | Authenticate with Infisical |
| `infisical init` | Initialize project |
| `infisical secrets` | List secrets |
| `infisical secrets set <key> <value>` | Set secret |
| `infisical secrets get <key>` | Get secret |
| `infisical run -- <command>` | Run command with secrets |
| `infisical export` | Export secrets |
| `infisical projects` | List projects |
| `infisical user` | Show current user |
| `infisical logout` | Logout from Infisical |

## Environment Variables

```bash
# Infisical configuration
export INFISICAL_TOKEN="your-service-token"
export INFISICAL_PROJECT_ID="your-project-id"
export INFISICAL_ENVIRONMENT="production"
export INFISICAL_DOMAIN="https://app.infisical.com"

# Debug and logging
export INFISICAL_DEBUG=true
export INFISICAL_LOG_LEVEL=debug

# CLI configuration
export INFISICAL_CONFIG_DIR="~/.config/infisical"
export INFISICAL_DISABLE_UPDATE_CHECK=true
```

## Best Practices

### Security Recommendations
1. **Rotate Service Tokens** - Regularly rotate service tokens
2. **Least Privilege** - Grant minimum necessary permissions
3. **Environment Separation** - Use separate projects for different environments
4. **Audit Logs** - Regularly review audit logs
5. **Backup Secrets** - Implement backup strategies
6. **Network Security** - Use HTTPS and proper network security
7. **Access Reviews** - Regularly review user access
8. **Token Management** - Securely store and manage tokens

### Development Workflow
1. **Local Development** - Use development environment for local work
2. **Git Integration** - Map Git branches to environments
3. **CI/CD Integration** - Automate secret injection in pipelines
4. **Secret Versioning** - Track secret changes and versions
5. **Documentation** - Document secret purposes and dependencies

---

*For more detailed information, visit the [official Infisical documentation](https://infisical.com/docs)*

