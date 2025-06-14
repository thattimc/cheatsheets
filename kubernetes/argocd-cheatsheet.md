# ArgoCD Cheatsheet

ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. It follows the GitOps pattern of using Git repositories as the source of truth for defining the desired application state.

## Overview

### Key Features
- **GitOps** - Git as the single source of truth
- **Declarative** - Kubernetes manifests define desired state
- **Automated Sync** - Automatic deployment from Git
- **Multi-cluster** - Deploy to multiple Kubernetes clusters
- **RBAC** - Role-based access control
- **SSO Integration** - OIDC, SAML, LDAP support
- **Web UI** - Rich visualization and management interface
- **CLI** - Command-line interface for automation
- **Webhook Support** - Git webhook integration
- **Health Monitoring** - Application health assessment

### Core Concepts
- **Application** - A group of Kubernetes resources defined in Git
- **Project** - A logical grouping of applications
- **Repository** - Git repository containing application manifests
- **Cluster** - Target Kubernetes cluster for deployment
- **Sync** - Process of applying Git state to cluster
- **Refresh** - Comparing Git state with live cluster state
- **Health** - Status of deployed resources

## Installation

### Install ArgoCD in Kubernetes
```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Access UI at https://localhost:8080
# Username: admin
# Password: <from above command>
```

### High Availability Installation
```bash
# Install HA version
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml

# Install with Helm
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd -n argocd --create-namespace
```

### Install ArgoCD CLI
```bash
# macOS
brew install argocd

# Linux
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

# Windows
choco install argocd-cli

# Verify installation
argocd version
```

## CLI Authentication and Configuration

### Login to ArgoCD
```bash
# Login with username/password
argocd login <ARGOCD_SERVER>
argocd login localhost:8080
argocd login argocd.example.com

# Login with token
argocd login <ARGOCD_SERVER> --username admin --password <PASSWORD>

# Login with SSO
argocd login <ARGOCD_SERVER> --sso

# Login with client certificates
argocd login <ARGOCD_SERVER> --client-crt /path/to/client.crt --client-crt-key /path/to/client.key

# Skip TLS verification (not recommended for production)
argocd login <ARGOCD_SERVER> --insecure

# Check login status
argocd account get-user-info
```

### Account Management
```bash
# Get current user info
argocd account get-user-info

# Update password
argocd account update-password

# Generate auth token
argocd account generate-token

# List contexts
argocd context

# Switch context
argocd context <CONTEXT_NAME>
```

## Application Management

### Create Applications
```bash
# Create application from CLI
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Create application with specific revision
argocd app create myapp \
  --repo https://github.com/myorg/myapp.git \
  --path manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace myapp \
  --revision v1.0.0

# Create application with Helm
argocd app create myapp \
  --repo https://github.com/myorg/myapp.git \
  --path helm-chart \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace myapp \
  --helm-set image.tag=v1.0.0

# Create application with Kustomize
argocd app create myapp \
  --repo https://github.com/myorg/myapp.git \
  --path overlays/production \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace myapp

# Create from YAML file
argocd app create -f application.yaml
```

### Application Operations
```bash
# List applications
argocd app list
argocd app list -o wide
argocd app list --selector env=production

# Get application details
argocd app get guestbook
argocd app get guestbook -o yaml
argocd app get guestbook --refresh

# Sync application
argocd app sync guestbook
argocd app sync guestbook --dry-run
argocd app sync guestbook --prune
argocd app sync guestbook --force

# Sync specific resources
argocd app sync guestbook --resource apps:Deployment:guestbook-ui

# Auto-sync configuration
argocd app set guestbook --sync-policy automated
argocd app set guestbook --auto-prune
argocd app set guestbook --self-heal

# Refresh application
argocd app refresh guestbook

# Delete application
argocd app delete guestbook
argocd app delete guestbook --cascade=false  # Keep resources

# Rollback application
argocd app rollback guestbook
argocd app rollback guestbook 123  # To specific revision
```

### Application Monitoring
```bash
# Get application history
argocd app history guestbook

# Get application resources
argocd app resources guestbook

# Get application logs
argocd app logs guestbook
argocd app logs guestbook --follow
argocd app logs guestbook --container mycontainer

# Wait for application sync
argocd app wait guestbook
argocd app wait guestbook --timeout 300

# Patch application
argocd app patch guestbook --patch '{"spec":{"source":{"targetRevision":"v2.0.0"}}}'

# Diff application
argocd app diff guestbook
```

## Application YAML Examples

### Basic Application
```yaml
# application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### Helm Application
```yaml
# helm-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-helm
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.bitnami.com/bitnami
    chart: nginx
    targetRevision: 9.5.0
    helm:
      parameters:
      - name: service.type
        value: LoadBalancer
      - name: replicaCount
        value: "3"
      values: |
        image:
          repository: nginx
          tag: 1.20.1
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
  destination:
    server: https://kubernetes.default.svc
    namespace: nginx
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### Kustomize Application
```yaml
# kustomize-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-kustomize
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp.git
    targetRevision: HEAD
    path: overlays/production
    kustomize:
      images:
      - myapp:v1.0.0
      commonLabels:
        environment: production
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp-prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### Multi-source Application
```yaml
# multi-source-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: multi-source-app
  namespace: argocd
spec:
  project: default
  sources:
  - repoURL: https://github.com/myorg/app-config.git
    targetRevision: HEAD
    path: .
  - repoURL: https://charts.bitnami.com/bitnami
    chart: postgresql
    targetRevision: 11.6.12
    helm:
      parameters:
      - name: auth.postgresPassword
        value: secretpassword
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Repository Management

### Repository Operations
```bash
# Add repository
argocd repo add https://github.com/myorg/myapp.git
argocd repo add https://github.com/myorg/myapp.git --username myuser --password mypass
argocd repo add https://github.com/myorg/myapp.git --ssh-private-key-path ~/.ssh/id_rsa

# Add Helm repository
argocd repo add https://charts.bitnami.com/bitnami --type helm --name bitnami

# List repositories
argocd repo list

# Get repository details
argocd repo get https://github.com/myorg/myapp.git

# Remove repository
argocd repo rm https://github.com/myorg/myapp.git
```

### Repository Credentials
```bash
# Add repository credentials
argocd repocreds add https://github.com/myorg --username myuser --password mypass

# Add SSH credentials
argocd repocreds add git@github.com:myorg --ssh-private-key-path ~/.ssh/id_rsa

# List repository credentials
argocd repocreds list

# Remove repository credentials
argocd repocreds rm https://github.com/myorg
```

## Cluster Management

### Cluster Operations
```bash
# Add cluster
argocd cluster add kubernetes-admin@kubernetes
argocd cluster add my-cluster --kubeconfig /path/to/kubeconfig

# List clusters
argocd cluster list

# Get cluster details
argocd cluster get https://kubernetes.default.svc

# Remove cluster
argocd cluster rm https://my-cluster.example.com

# Rotate cluster credentials
argocd cluster rotate-auth https://kubernetes.default.svc
```

### Service Account for ArgoCD
```yaml
# argocd-service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: argocd-manager
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: argocd-manager-role
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argocd-manager-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argocd-manager-role
subjects:
- kind: ServiceAccount
  name: argocd-manager
  namespace: kube-system
```

## Project Management

### Project Operations
```bash
# Create project
argocd proj create myproject

# List projects
argocd proj list

# Get project details
argocd proj get myproject

# Delete project
argocd proj delete myproject

# Add source repository to project
argocd proj add-source myproject https://github.com/myorg/myapp.git

# Add destination to project
argocd proj add-destination myproject https://kubernetes.default.svc myapp

# Set project roles
argocd proj role create myproject developer
argocd proj role add-policy myproject developer -a allow -r "myproject/*" -o "*"
```

### Project YAML
```yaml
# project.yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: myproject
  namespace: argocd
spec:
  description: My application project
  sourceRepos:
  - 'https://github.com/myorg/*'
  - 'https://charts.bitnami.com/bitnami'
  destinations:
  - namespace: 'myapp-*'
    server: https://kubernetes.default.svc
  - namespace: 'myapp-*'
    server: https://prod-cluster.example.com
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  - group: 'rbac.authorization.k8s.io'
    kind: ClusterRole
  namespaceResourceWhitelist:
  - group: '*'
    kind: '*'
  roles:
  - name: developer
    description: Developer role
    policies:
    - p, proj:myproject:developer, applications, *, myproject/*, allow
    - p, proj:myproject:developer, repositories, *, *, allow
    groups:
    - myorg:developers
  - name: admin
    description: Admin role
    policies:
    - p, proj:myproject:admin, *, *, myproject/*, allow
    groups:
    - myorg:admins
```

## ConfigMaps and Configuration

### ArgoCD Configuration
```yaml
# argocd-cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # Application instance label key
  application.instanceLabelKey: argocd.argoproj.io/instance
  
  # Server properties
  server.insecure: "true"
  
  # OIDC configuration
  oidc.config: |
    name: OIDC
    issuer: https://auth.example.com
    clientId: argocd
    clientSecret: $oidc.clientSecret
    requestedScopes: ["openid", "profile", "email", "groups"]
    requestedIDTokenClaims: {"groups": {"essential": true}}
  
  # Git webhook configuration
  webhook.github.secret: $webhook.github.secret
  
  # Resource customizations
  resource.customizations: |
    networking.k8s.io/Ingress:
      health.lua: |
        hs = {}
        hs.status = "Healthy"
        return hs
  
  # Repository credentials template
  repository.credentials: |
    - url: https://github.com/myorg
      usernameSecret:
        name: github-creds
        key: username
      passwordSecret:
        name: github-creds
        key: password
```

### RBAC Configuration
```yaml
# argocd-rbac-cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly
  policy.csv: |
    p, role:admin, applications, *, */*, allow
    p, role:admin, clusters, *, *, allow
    p, role:admin, repositories, *, *, allow
    p, role:developer, applications, *, default/*, allow
    p, role:developer, applications, sync, default/*, allow
    g, myorg:admins, role:admin
    g, myorg:developers, role:developer
```

## Sync Policies and Options

### Automated Sync Policy
```yaml
# Automated sync with prune and self-heal
syncPolicy:
  automated:
    prune: true      # Delete resources not in Git
    selfHeal: true   # Revert manual changes
  syncOptions:
  - CreateNamespace=true
  - PrunePropagationPolicy=foreground
  - PruneLast=true
  - RespectIgnoreDifferences=true
  - ApplyOutOfSyncOnly=true
```

### Manual Sync Policy
```yaml
# Manual sync only
syncPolicy:
  syncOptions:
  - CreateNamespace=true
  - Validate=false
  - SkipDryRunOnMissingResource=true
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

### Sync Windows
```yaml
# project.yaml with sync windows
spec:
  syncWindows:
  - kind: allow
    schedule: "0 22 * * *"  # 10 PM daily
    duration: 1h
    applications:
    - "*"
    manualSync: true
  - kind: deny
    schedule: "0 0 * * 0"   # Sunday midnight
    duration: 24h
    applications:
    - "production-*"
```

## Health Checks and Resource Hooks

### Custom Health Checks
```yaml
# In argocd-cm ConfigMap
resource.customizations.health.argoproj.io_Rollout: |
  hs = {}
  if obj.status ~= nil then
    if obj.status.phase == "Degraded" then
      hs.status = "Degraded"
      hs.message = obj.status.message
      return hs
    end
    if obj.status.phase == "Progressing" then
      hs.status = "Progressing"
      hs.message = obj.status.message
      return hs
    end
  end
  hs.status = "Healthy"
  return hs
```

### Resource Hooks
```yaml
# pre-sync-hook.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: database-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
    argocd.argoproj.io/sync-wave: "1"
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: migrate/migrate
        command: ["migrate", "-path", "/migrations", "-database", "postgres://...", "up"]
      restartPolicy: Never
  backoffLimit: 0
```

### Sync Waves
```yaml
# Apply resources in order using sync waves
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
  annotations:
    argocd.argoproj.io/sync-wave: "1"  # Applied first
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  annotations:
    argocd.argoproj.io/sync-wave: "2"  # Applied second
```

## Notifications

### Notification Configuration
```yaml
# argocd-notifications-cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.slack: |
    token: $slack-token
  service.email: |
    host: smtp.gmail.com
    port: 587
    from: argocd@example.com
    username: $email-username
    password: $email-password
  template.app-deployed: |
    email:
      subject: New version of an application {{.app.metadata.name}} is up and running.
    message: |
      {{if eq .serviceType "slack"}}:white_check_mark:{{end}} Application {{.app.metadata.name}} is now running new version.
    slack:
      attachments: |
        [{
          "title": "{{ .app.metadata.name}}",
          "title_link":"{{.context.argocdUrl}}/applications/{{.app.metadata.name}}",
          "color": "#18be52",
          "fields": [
          {
            "title": "Sync Status",
            "value": "{{.app.status.sync.status}}",
            "short": true
          },
          {
            "title": "Repository",
            "value": "{{.app.spec.source.repoURL}}",
            "short": true
          },
          {
            "title": "Revision",
            "value": "{{.app.status.sync.revision}}",
            "short": true
          }
          ]
        }]
  trigger.on-deployed: |
    - when: app.status.operationState.phase in ['Succeeded'] and app.status.health.status == 'Healthy'
      send: [app-deployed]
  subscriptions: |
    - recipients:
      - slack:general
      triggers:
      - on-deployed
```

### Application Annotations for Notifications
```yaml
# Add to application metadata
metadata:
  annotations:
    notifications.argoproj.io/subscribe.on-sync-succeeded.slack: general
    notifications.argoproj.io/subscribe.on-health-degraded.email: team@example.com
    notifications.argoproj.io/subscribe.on-sync-failed.slack: alerts
```

## ArgoCD CLI Commands Reference

### Application Commands
```bash
# Core application operations
argocd app create <APP_NAME>           # Create application
argocd app list                        # List applications
argocd app get <APP_NAME>              # Get application details
argocd app sync <APP_NAME>             # Sync application
argocd app delete <APP_NAME>           # Delete application
argocd app diff <APP_NAME>             # Show diff
argocd app rollback <APP_NAME>         # Rollback application
argocd app history <APP_NAME>          # Show sync history
argocd app resources <APP_NAME>        # Show resources
argocd app logs <APP_NAME>             # Show logs
argocd app wait <APP_NAME>             # Wait for sync

# Application configuration
argocd app set <APP_NAME>              # Update application
argocd app patch <APP_NAME>            # Patch application
argocd app edit <APP_NAME>             # Edit application

# Resource operations
argocd app manifests <APP_NAME>        # Show manifests
argocd app terminate-op <APP_NAME>     # Terminate operation
argocd app patch-resource <APP_NAME>   # Patch resource
```

### Repository Commands
```bash
argocd repo add <REPO_URL>             # Add repository
argocd repo list                       # List repositories
argocd repo get <REPO_URL>             # Get repository
argocd repo rm <REPO_URL>              # Remove repository
```

### Cluster Commands
```bash
argocd cluster add <CONTEXT>           # Add cluster
argocd cluster list                    # List clusters
argocd cluster get <SERVER>            # Get cluster
argocd cluster rm <SERVER>             # Remove cluster
```

### Project Commands
```bash
argocd proj create <PROJECT>           # Create project
argocd proj list                       # List projects
argocd proj get <PROJECT>              # Get project
argocd proj delete <PROJECT>           # Delete project
argocd proj add-source <PROJECT> <URL> # Add source repo
argocd proj add-destination <PROJECT>  # Add destination
```

## GitOps Workflow Examples

### Basic GitOps Workflow
```bash
# 1. Developer pushes code to Git
git add .
git commit -m "feat: add new feature"
git push origin main

# 2. CI pipeline builds and pushes image
docker build -t myapp:v1.2.3 .
docker push myregistry/myapp:v1.2.3

# 3. Update manifest repository
git clone https://github.com/myorg/myapp-manifests.git
cd myapp-manifests
sed -i 's/myapp:v1.2.2/myapp:v1.2.3/g' deployment.yaml
git add .
git commit -m "chore: update image to v1.2.3"
git push origin main

# 4. ArgoCD automatically syncs (if auto-sync enabled)
# Or manually sync
argocd app sync myapp
```

### Multi-environment Workflow
```bash
# Development environment
argocd app create myapp-dev \
  --repo https://github.com/myorg/myapp-manifests.git \
  --path overlays/dev \
  --dest-namespace myapp-dev

# Staging environment
argocd app create myapp-staging \
  --repo https://github.com/myorg/myapp-manifests.git \
  --path overlays/staging \
  --dest-namespace myapp-staging

# Production environment (manual sync)
argocd app create myapp-prod \
  --repo https://github.com/myorg/myapp-manifests.git \
  --path overlays/prod \
  --dest-namespace myapp-prod \
  --sync-policy none
```

## Monitoring and Observability

### Application Metrics
```bash
# Get application sync status
argocd app get myapp --output json | jq '.status.sync.status'

# Get application health
argocd app get myapp --output json | jq '.status.health.status'

# Monitor sync operations
watch -n 5 'argocd app list'

# Get detailed resource status
argocd app resources myapp --output wide
```

### Prometheus Metrics
```yaml
# ServiceMonitor for ArgoCD metrics
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-metrics
  namespace: argocd
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-metrics
  endpoints:
  - port: metrics
```

### Grafana Dashboard
```json
{
  "dashboard": {
    "title": "ArgoCD",
    "panels": [
      {
        "title": "Applications by Status",
        "type": "stat",
        "targets": [
          {
            "expr": "sum by (health_status) (argocd_app_health_status)"
          }
        ]
      },
      {
        "title": "Sync Activity",
        "type": "graph",
        "targets": [
          {
            "expr": "increase(argocd_app_sync_total[5m])"
          }
        ]
      }
    ]
  }
}
```

## Security Best Practices

### RBAC Configuration
```yaml
# Minimal permissions for developer role
policy.csv: |
  p, role:developer, applications, get, */*, allow
  p, role:developer, applications, sync, myproject/*, allow
  p, role:developer, repositories, get, *, allow
  p, role:developer, logs, get, myproject/*, allow
  g, myorg:developers, role:developer
```

### Secure Repository Access
```bash
# Use SSH keys for private repositories
ssh-keygen -t ed25519 -f ~/.ssh/argocd_key -N ""

# Add SSH key to ArgoCD
argocd repo add git@github.com:myorg/myapp.git \
  --ssh-private-key-path ~/.ssh/argocd_key

# Use repository credentials template
kubectl create secret generic github-creds \
  --from-literal=username=myuser \
  --from-literal=password=mytoken \
  -n argocd
```

### Network Security
```yaml
# NetworkPolicy for ArgoCD
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: argocd-network-policy
  namespace: argocd
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: argocd-server
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to: []
    ports:
    - protocol: TCP
      port: 443  # HTTPS to Git repositories
    - protocol: TCP
      port: 6443 # Kubernetes API
```

## Troubleshooting

### Common Issues
```bash
# Application stuck in sync
argocd app get myapp --refresh
argocd app sync myapp --force

# Permission denied errors
kubectl get clusterrolebindings | grep argocd
kubectl describe clusterrolebinding argocd-server

# Repository connection issues
argocd repo get https://github.com/myorg/myapp.git
kubectl logs -n argocd deployment/argocd-repo-server

# Application not syncing automatically
argocd app get myapp -o yaml | grep -A 10 syncPolicy

# Check ArgoCD server logs
kubectl logs -n argocd deployment/argocd-server -f

# Check application controller logs
kubectl logs -n argocd deployment/argocd-application-controller -f

# Validate application manifest
argocd app manifests myapp --local-repo-root ./

# Debug sync operation
argocd app sync myapp --dry-run --local-repo-root ./
```

### Resource Conflicts
```bash
# Force replace resources
argocd app sync myapp --force

# Ignore differences
annotations:
  argocd.argoproj.io/compare-options: IgnoreExtraneous
  argocd.argoproj.io/sync-options: Replace=true

# Skip dry run for missing resources
annotations:
  argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
```

### Performance Tuning
```yaml
# argocd-cm.yaml
data:
  # Increase timeout for large applications
  timeout.reconciliation: 180s
  timeout.hard.reconciliation: 0
  
  # Adjust resource tracking
  application.resourceTrackingMethod: annotation
  
  # Configure repository cache
  reposerver.parallelism.limit: "10"
```

## Advanced Configuration

### Custom Tools
```yaml
# argocd-cm.yaml
data:
  configManagementPlugins: |
    - name: helm-secrets
      generate:
        command: ["helm"]
        args: ["secrets", "template", ".", "-f", "values.yaml"]
    - name: kustomize-sops
      generate:
        command: ["sh", "-c"]
        args: ["sops -d secrets.yaml | kustomize build"]
```

### Resource Exclusions
```yaml
# In Application spec
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas
  - group: ""
    kind: Secret
    name: mysecret
    jsonPointers:
    - /data
```

### Custom Health Checks
```yaml
# argocd-cm.yaml
data:
  resource.customizations.health.networking.k8s.io_Ingress: |
    hs = {}
    if obj.status ~= nil then
      if obj.status.loadBalancer ~= nil then
        if obj.status.loadBalancer.ingress ~= nil then
          hs.status = "Healthy"
          hs.message = "Ingress has been assigned an IP/hostname"
          return hs
        end
      end
    end
    hs.status = "Progressing"
    hs.message = "Waiting for ingress to be assigned an IP/hostname"
    return hs
```

## Useful Environment Variables

### ArgoCD CLI
```bash
# Server configuration
export ARGOCD_SERVER=argocd.example.com
export ARGOCD_AUTH_TOKEN=<auth-token>
export ARGOCD_OPTS="--grpc-web --insecure"

# Default context
export ARGOCD_CONTEXT=argocd.example.com

# Output format
export ARGOCD_OUTPUT=json

# Client configuration
export ARGOCD_CLIENT_CERT=/path/to/client.crt
export ARGOCD_CLIENT_CERT_KEY=/path/to/client.key
```

### ArgoCD Server
```bash
# Server configuration
ARGOCD_SERVER_INSECURE=true
ARGOCD_SERVER_ROOTPATH=/argocd
ARGOCD_SERVER_GRPC_WEB=true

# OIDC configuration
ARGOCD_OIDC_ISSUER_ROOT_CA=/etc/ssl/certs/ca.pem
ARGOCD_OIDC_TLS_INSECURE_SKIP_VERIFY=false

# Application configuration
ARGOCD_APPLICATION_NAMESPACES=argocd,apps
```

## Integration Examples

### GitHub Actions Integration
```yaml
# .github/workflows/deploy.yml
name: Deploy to ArgoCD
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup ArgoCD CLI
      run: |
        curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
        chmod +x /usr/local/bin/argocd
    
    - name: Login to ArgoCD
      run: |
        argocd login ${{ secrets.ARGOCD_SERVER }} --username ${{ secrets.ARGOCD_USERNAME }} --password ${{ secrets.ARGOCD_PASSWORD }}
    
    - name: Sync Application
      run: |
        argocd app sync myapp --timeout 300
        argocd app wait myapp --timeout 300
```

### Terraform Integration
```hcl
# argocd.tf
resource "argocd_application" "myapp" {
  metadata {
    name      = "myapp"
    namespace = "argocd"
  }
  
  spec {
    source {
      repo_url        = "https://github.com/myorg/myapp.git"
      path            = "manifests"
      target_revision = "HEAD"
    }
    
    destination {
      server    = "https://kubernetes.default.svc"
      namespace = "myapp"
    }
    
    sync_policy {
      automated {
        prune     = true
        self_heal = true
      }
      sync_options = ["CreateNamespace=true"]
    }
  }
}
```

---

*For more detailed information, visit the [official ArgoCD documentation](https://argo-cd.readthedocs.io/)*

