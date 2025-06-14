# kubectl Cheatsheet

kubectl is the command-line tool for interacting with Kubernetes clusters. This cheatsheet covers the most commonly used commands and operations.

## Installation

### macOS
```bash
# Install via Homebrew
brew install kubectl

# Or download binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### Linux
```bash
# Download latest release
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install via package manager (Ubuntu/Debian)
sudo apt-get update && sudo apt-get install -y kubectl
```

### Windows
```powershell
# Using winget
winget install Kubernetes.kubectl

# Using Chocolatey
choco install kubernetes-cli
```

## Basic Commands

### Cluster Information
```bash
# Check kubectl version
kubectl version --client
kubectl version --short

# Get cluster information
kubectl cluster-info
kubectl cluster-info dump

# Get cluster configuration
kubectl config view
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Set default namespace
kubectl config set-context --current --namespace=<namespace>
```

### Get Resources
```bash
# Get all resources
kubectl get all
kubectl get all --all-namespaces
kubectl get all -A  # Short for --all-namespaces

# Get specific resources
kubectl get pods
kubectl get services
kubectl get deployments
kubectl get nodes

# Get resources with more details
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -o json

# Get resources from specific namespace
kubectl get pods -n <namespace>
kubectl get pods --namespace=<namespace>

# Get resources with labels
kubectl get pods --show-labels
kubectl get pods -l app=nginx
kubectl get pods --selector=app=nginx
```

## Pods

### Pod Operations
```bash
# List pods
kubectl get pods
kubectl get po  # Short alias

# Get pod details
kubectl describe pod <pod-name>
kubectl describe po <pod-name>

# Get pod YAML
kubectl get pod <pod-name> -o yaml

# Get pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container pod
kubectl logs <pod-name> --previous  # Previous container logs
kubectl logs <pod-name> -f  # Follow logs
kubectl logs <pod-name> --tail=100  # Last 100 lines

# Execute commands in pod
kubectl exec <pod-name> -- <command>
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh

# Port forwarding
kubectl port-forward <pod-name> 8080:80
kubectl port-forward pod/<pod-name> 8080:80

# Copy files to/from pod
kubectl cp <local-file> <pod-name>:<remote-path>
kubectl cp <pod-name>:<remote-path> <local-file>
```

### Pod Creation and Management
```bash
# Create pod from image
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --port=80

# Create pod with dry run (generate YAML)
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Delete pod
kubectl delete pod <pod-name>
kubectl delete po <pod-name>

# Delete all pods
kubectl delete pods --all

# Delete pods by label
kubectl delete pods -l app=nginx
```

## Deployments

### Deployment Operations
```bash
# List deployments
kubectl get deployments
kubectl get deploy  # Short alias

# Get deployment details
kubectl describe deployment <deployment-name>
kubectl describe deploy <deployment-name>

# Create deployment
kubectl create deployment nginx --image=nginx
kubectl create deployment nginx --image=nginx --replicas=3

# Generate deployment YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml

# Scale deployment
kubectl scale deployment <deployment-name> --replicas=5
kubectl scale deploy <deployment-name> --replicas=5

# Update deployment image
kubectl set image deployment/<deployment-name> <container-name>=<new-image>
kubectl set image deploy/nginx nginx=nginx:1.20

# Rollout operations
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name> --to-revision=2

# Delete deployment
kubectl delete deployment <deployment-name>
kubectl delete deploy <deployment-name>
```

## Services

### Service Operations
```bash
# List services
kubectl get services
kubectl get svc  # Short alias

# Get service details
kubectl describe service <service-name>
kubectl describe svc <service-name>

# Create service
kubectl expose deployment <deployment-name> --type=ClusterIP --port=80
kubectl expose deployment <deployment-name> --type=NodePort --port=80
kubectl expose deployment <deployment-name> --type=LoadBalancer --port=80

# Create service from pod
kubectl expose pod <pod-name> --port=80 --target-port=8080

# Port forward to service
kubectl port-forward service/<service-name> 8080:80
kubectl port-forward svc/<service-name> 8080:80

# Delete service
kubectl delete service <service-name>
kubectl delete svc <service-name>
```

## ConfigMaps and Secrets

### ConfigMaps
```bash
# List configmaps
kubectl get configmaps
kubectl get cm  # Short alias

# Create configmap from literal values
kubectl create configmap <cm-name> --from-literal=key1=value1 --from-literal=key2=value2

# Create configmap from file
kubectl create configmap <cm-name> --from-file=<file-path>
kubectl create configmap <cm-name> --from-file=<key>=<file-path>

# Create configmap from directory
kubectl create configmap <cm-name> --from-file=<directory-path>

# Get configmap details
kubectl describe configmap <cm-name>
kubectl get configmap <cm-name> -o yaml

# Delete configmap
kubectl delete configmap <cm-name>
```

### Secrets
```bash
# List secrets
kubectl get secrets

# Create secret from literal values
kubectl create secret generic <secret-name> --from-literal=username=admin --from-literal=password=secret

# Create secret from files
kubectl create secret generic <secret-name> --from-file=<file-path>

# Create TLS secret
kubectl create secret tls <secret-name> --cert=<cert-file> --key=<key-file>

# Create docker registry secret
kubectl create secret docker-registry <secret-name> \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>

# Get secret details
kubectl describe secret <secret-name>
kubectl get secret <secret-name> -o yaml

# Decode secret value
kubectl get secret <secret-name> -o jsonpath='{.data.<key>}' | base64 --decode

# Delete secret
kubectl delete secret <secret-name>
```

## Namespaces

### Namespace Operations
```bash
# List namespaces
kubectl get namespaces
kubectl get ns  # Short alias

# Create namespace
kubectl create namespace <namespace-name>
kubectl create ns <namespace-name>

# Get namespace details
kubectl describe namespace <namespace-name>

# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace-name>

# Get resources from specific namespace
kubectl get pods -n <namespace-name>
kubectl get all -n <namespace-name>

# Get resources from all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# Delete namespace (and all resources in it)
kubectl delete namespace <namespace-name>
```

## YAML Manifests

### Apply and Manage Manifests
```bash
# Apply YAML file
kubectl apply -f <file.yaml>
kubectl apply -f <directory>/
kubectl apply -f <url>

# Apply with dry run
kubectl apply -f <file.yaml> --dry-run=client
kubectl apply -f <file.yaml> --dry-run=server

# Delete from YAML file
kubectl delete -f <file.yaml>

# Create from YAML file
kubectl create -f <file.yaml>

# Replace resource
kubectl replace -f <file.yaml>

# Validate YAML file
kubectl apply -f <file.yaml> --validate=true --dry-run=client

# Apply and record the command
kubectl apply -f <file.yaml> --record
```

### Generate YAML Templates
```bash
# Generate pod YAML
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Generate deployment YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Generate service YAML
kubectl expose deployment nginx --port=80 --dry-run=client -o yaml > service.yaml

# Generate configmap YAML
kubectl create configmap myconfig --from-literal=key=value --dry-run=client -o yaml > configmap.yaml
```

## Resource Management

### Labels and Annotations
```bash
# Add label to resource
kubectl label pod <pod-name> env=production
kubectl label deployment <deployment-name> version=v1.0

# Remove label
kubectl label pod <pod-name> env-

# Update label
kubectl label pod <pod-name> env=staging --overwrite

# Add annotation
kubectl annotate pod <pod-name> description="This is a test pod"

# Remove annotation
kubectl annotate pod <pod-name> description-

# Get resources by label
kubectl get pods -l env=production
kubectl get pods --selector=env=production,version=v1.0
```

### Resource Quotas and Limits
```bash
# Get resource quotas
kubectl get resourcequotas
kubectl get quota

# Describe resource quota
kubectl describe resourcequota <quota-name>

# Get limit ranges
kubectl get limitranges
kubectl get limits

# Describe limit range
kubectl describe limitrange <limit-name>
```

## Troubleshooting

### Debugging Commands
```bash
# Get events
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.name=<pod-name>

# Describe resources for debugging
kubectl describe pod <pod-name>
kubectl describe node <node-name>
kubectl describe service <service-name>

# Check resource usage
kubectl top nodes
kubectl top pods
kubectl top pods --containers

# Get pod logs for debugging
kubectl logs <pod-name> --previous
kubectl logs <pod-name> -f
kubectl logs <pod-name> --since=1h

# Debug pod networking
kubectl exec -it <pod-name> -- nslookup <service-name>
kubectl exec -it <pod-name> -- wget -qO- <service-name>

# Debug with temporary pod
kubectl run debug --image=busybox --rm -it --restart=Never -- /bin/sh
```

### Pod States and Conditions
```bash
# Check pod status
kubectl get pods -o wide
kubectl get pods --field-selector=status.phase=Failed
kubectl get pods --field-selector=status.phase=Pending

# Get pod conditions
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,CONDITIONS:.status.conditions[*].type
```

## Advanced Operations

### JSONPath and Custom Columns
```bash
# Use JSONPath to extract specific data
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{.items[*].status.podIP}'
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP
kubectl get nodes -o custom-columns=NAME:.metadata.name,STATUS:.status.conditions[-1].type
```

### Watch and Wait
```bash
# Watch resources for changes
kubectl get pods --watch
kubectl get pods -w

# Wait for condition
kubectl wait --for=condition=ready pod/<pod-name>
kubectl wait --for=condition=available deployment/<deployment-name>
kubectl wait --for=delete pod/<pod-name> --timeout=60s
```

### Patch Resources
```bash
# Strategic merge patch
kubectl patch deployment <deployment-name> -p '{"spec":{"replicas":3}}'

# JSON merge patch
kubectl patch deployment <deployment-name> --type='merge' -p '{"spec":{"replicas":3}}'

# JSON patch
kubectl patch deployment <deployment-name> --type='json' -p='[{"op": "replace", "path": "/spec/replicas", "value": 3}]'
```

## Contexts and Clusters

### Context Management
```bash
# List contexts
kubectl config get-contexts

# Current context
kubectl config current-context

# Switch context
kubectl config use-context <context-name>

# Set cluster
kubectl config set-cluster <cluster-name> --server=<server-url>

# Set credentials
kubectl config set-credentials <user-name> --token=<token>

# Set context
kubectl config set-context <context-name> --cluster=<cluster-name> --user=<user-name>

# Delete context
kubectl config delete-context <context-name>
```

## Useful kubectl Aliases

```bash
# Add these to your shell profile (.bashrc, .zshrc, etc.)
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias ka='kubectl apply'
alias kdel='kubectl delete'

# Resource aliases
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kgn='kubectl get nodes'

# Namespace aliases
alias kgpn='kubectl get pods --all-namespaces'
alias kgsn='kubectl get services --all-namespaces'

# Logs and exec aliases
alias kl='kubectl logs'
alias ke='kubectl exec -it'
```

## Quick Reference Table

| Resource | Short Name | kubectl Command |
|----------|------------|----------------|
| **Pods** | `po` | `kubectl get pods` |
| **Services** | `svc` | `kubectl get services` |
| **Deployments** | `deploy` | `kubectl get deployments` |
| **ReplicaSets** | `rs` | `kubectl get replicasets` |
| **ConfigMaps** | `cm` | `kubectl get configmaps` |
| **Secrets** | | `kubectl get secrets` |
| **Namespaces** | `ns` | `kubectl get namespaces` |
| **Nodes** | `no` | `kubectl get nodes` |
| **PersistentVolumes** | `pv` | `kubectl get persistentvolumes` |
| **PersistentVolumeClaims** | `pvc` | `kubectl get persistentvolumeclaims` |
| **Ingress** | `ing` | `kubectl get ingress` |
| **StorageClass** | `sc` | `kubectl get storageclasses` |

## Common kubectl Patterns

### One-liners
```bash
# Get all pod IPs
kubectl get pods -o jsonpath='{.items[*].status.podIP}'

# Get all node names
kubectl get nodes -o jsonpath='{.items[*].metadata.name}'

# Delete all failed pods
kubectl delete pods --field-selector=status.phase=Failed

# Delete all evicted pods
kubectl get pods --all-namespaces --field-selector=status.phase=Failed -o json | kubectl delete -f -

# Get pods using most CPU
kubectl top pods --sort-by=cpu

# Get pods using most memory
kubectl top pods --sort-by=memory

# Force delete pod
kubectl delete pod <pod-name> --grace-period=0 --force

# Get all images used in cluster
kubectl get pods -o jsonpath="{.items[*].spec.containers[*].image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
```

## Sample YAML Templates

### Basic Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Basic Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Basic Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

## Environment Setup

### kubectl Autocompletion
```bash
# For bash
echo 'source <(kubectl completion bash)' >>~/.bashrc

# For zsh
echo 'source <(kubectl completion zsh)' >>~/.zshrc

# For fish
kubectl completion fish | source
```

### Useful Tools
```bash
# Install kubectx and kubens for easier context/namespace switching
brew install kubectx

# Switch contexts
kubectx <context-name>

# Switch namespaces
kubens <namespace-name>

# Install k9s for terminal UI
brew install k9s
k9s
```

---

*For more detailed information, visit the [official Kubernetes documentation](https://kubernetes.io/docs/reference/kubectl/)*

