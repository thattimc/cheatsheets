# Azure CLI Cheatsheet

## Installation & Setup

### Install Azure CLI
```bash
# macOS
brew install azure-cli

# Linux (Ubuntu/Debian)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Linux (RHEL/CentOS)
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[azure-cli]\nname=Azure CLI\nbaseurl=https://packages.microsoft.com/yumrepos/azure-cli\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/azure-cli.repo'
sudo yum install azure-cli

# Windows
winget install -e --id Microsoft.AzureCLI

# Python pip
pip install azure-cli

# Docker
docker run -it mcr.microsoft.com/azure-cli
```

### Authentication & Login
```bash
# Interactive login
az login

# Login with service principal
az login --service-principal --username APP_ID --password PASSWORD --tenant TENANT_ID

# Login with managed identity
az login --identity

# Login to specific cloud
az login --cloud AzureGermanCloud
az login --cloud AzureChinaCloud
az login --cloud AzureUSGovernment

# Check current account
az account show

# List available subscriptions
az account list

# Set active subscription
az account set --subscription "subscription-id-or-name"

# Logout
az logout
```

### Configuration
```bash
# Show current configuration
az configure
az config get

# Set default values
az config set defaults.location=eastus
az config set defaults.group=myResourceGroup
az config set core.output=table

# List configuration values
az config get core.output
az config get defaults.location

# Reset configuration
az config unset defaults.location
```

## Resource Groups

### Basic Operations
```bash
# List resource groups
az group list
az group list --output table

# Create resource group
az group create --name myResourceGroup --location eastus

# Show resource group details
az group show --name myResourceGroup

# Delete resource group
az group delete --name myResourceGroup --yes --no-wait

# Check if resource group exists
az group exists --name myResourceGroup

# List resources in group
az resource list --resource-group myResourceGroup
```

### Resource Group Management
```bash
# Update resource group tags
az group update --name myResourceGroup --tags Environment=Production Team=Backend

# Export resource group template
az group export --name myResourceGroup --output-template template.json

# Validate resource group deployment
az deployment group validate --resource-group myResourceGroup --template-file template.json

# Deploy to resource group
az deployment group create --resource-group myResourceGroup --template-file template.json
```

## Virtual Machines

### VM Operations
```bash
# List VMs
az vm list
az vm list --resource-group myResourceGroup --output table

# Create VM
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys

# Create Windows VM
az vm create \
  --resource-group myResourceGroup \
  --name myWindowsVM \
  --image Win2022Datacenter \
  --admin-username azureuser \
  --admin-password MyPassword123!

# Start/Stop/Restart VM
az vm start --resource-group myResourceGroup --name myVM
az vm stop --resource-group myResourceGroup --name myVM
az vm restart --resource-group myResourceGroup --name myVM

# Deallocate VM (stop billing)
az vm deallocate --resource-group myResourceGroup --name myVM

# Delete VM
az vm delete --resource-group myResourceGroup --name myVM --yes
```

### VM Information
```bash
# Show VM details
az vm show --resource-group myResourceGroup --name myVM

# Get VM sizes available in location
az vm list-sizes --location eastus

# List VM images
az vm image list --output table
az vm image list --publisher Canonical --output table

# Get VM IP addresses
az vm list-ip-addresses --resource-group myResourceGroup --name myVM

# Get VM status
az vm get-instance-view --resource-group myResourceGroup --name myVM
```

### VM Extensions
```bash
# List available extensions
az vm extension image list --output table

# Install extension
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name CustomScript \
  --publisher Microsoft.Azure.Extensions

# List installed extensions
az vm extension list --resource-group myResourceGroup --vm-name myVM
```

## Storage Accounts

### Storage Account Management
```bash
# List storage accounts
az storage account list
az storage account list --resource-group myResourceGroup

# Create storage account
az storage account create \
  --resource-group myResourceGroup \
  --name mystorageaccount \
  --location eastus \
  --sku Standard_LRS

# Show storage account
az storage account show --name mystorageaccount --resource-group myResourceGroup

# Delete storage account
az storage account delete --name mystorageaccount --resource-group myResourceGroup --yes

# Get storage account keys
az storage account keys list --account-name mystorageaccount --resource-group myResourceGroup

# Regenerate storage account key
az storage account keys renew --account-name mystorageaccount --key key1
```

### Blob Storage
```bash
# Create container
az storage container create --name mycontainer --account-name mystorageaccount

# List containers
az storage container list --account-name mystorageaccount

# Upload blob
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --file ./local-file.txt

# Download blob
az storage blob download \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --file ./downloaded-file.txt

# List blobs
az storage blob list --account-name mystorageaccount --container-name mycontainer

# Delete blob
az storage blob delete --account-name mystorageaccount --container-name mycontainer --name myblob.txt

# Generate SAS token
az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --permissions r \
  --expiry 2024-12-31T23:59Z
```

## App Service

### App Service Plans
```bash
# List app service plans
az appservice plan list

# Create app service plan
az appservice plan create \
  --resource-group myResourceGroup \
  --name myAppServicePlan \
  --location eastus \
  --sku B1

# Show app service plan
az appservice plan show --name myAppServicePlan --resource-group myResourceGroup

# Update app service plan
az appservice plan update --name myAppServicePlan --resource-group myResourceGroup --sku S1

# Delete app service plan
az appservice plan delete --name myAppServicePlan --resource-group myResourceGroup --yes
```

### Web Apps
```bash
# List web apps
az webapp list
az webapp list --resource-group myResourceGroup

# Create web app
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myWebApp \
  --runtime "PYTHON|3.9"

# Show web app
az webapp show --name myWebApp --resource-group myResourceGroup

# Start/Stop/Restart web app
az webapp start --name myWebApp --resource-group myResourceGroup
az webapp stop --name myWebApp --resource-group myResourceGroup
az webapp restart --name myWebApp --resource-group myResourceGroup

# Delete web app
az webapp delete --name myWebApp --resource-group myResourceGroup

# Deploy from GitHub
az webapp deployment source config \
  --name myWebApp \
  --resource-group myResourceGroup \
  --repo-url https://github.com/user/repo \
  --branch main
```

### Web App Configuration
```bash
# List app settings
az webapp config appsettings list --name myWebApp --resource-group myResourceGroup

# Set app settings
az webapp config appsettings set \
  --name myWebApp \
  --resource-group myResourceGroup \
  --settings KEY1=VALUE1 KEY2=VALUE2

# Delete app setting
az webapp config appsettings delete \
  --name myWebApp \
  --resource-group myResourceGroup \
  --setting-names KEY1

# Show connection strings
az webapp config connection-string list --name myWebApp --resource-group myResourceGroup

# Set connection string
az webapp config connection-string set \
  --name myWebApp \
  --resource-group myResourceGroup \
  --connection-string-type SQLServer \
  --settings MyDB="Server=myserver;Database=mydb;User ID=myuser;Password=mypass;"
```

## Azure Functions

### Function Apps
```bash
# List function apps
az functionapp list

# Create function app
az functionapp create \
  --resource-group myResourceGroup \
  --consumption-plan-location eastus \
  --runtime python \
  --runtime-version 3.9 \
  --functions-version 4 \
  --name myFunctionApp \
  --storage-account mystorageaccount

# Show function app
az functionapp show --name myFunctionApp --resource-group myResourceGroup

# Start/Stop function app
az functionapp start --name myFunctionApp --resource-group myResourceGroup
az functionapp stop --name myFunctionApp --resource-group myResourceGroup

# Delete function app
az functionapp delete --name myFunctionApp --resource-group myResourceGroup

# Deploy function app
az functionapp deployment source config-zip \
  --name myFunctionApp \
  --resource-group myResourceGroup \
  --src function-app.zip
```

## Azure Container Instances (ACI)

### Container Management
```bash
# List container groups
az container list

# Create container
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --dns-name-label aci-demo \
  --ports 80

# Show container
az container show --resource-group myResourceGroup --name mycontainer

# Get container logs
az container logs --resource-group myResourceGroup --name mycontainer

# Execute command in container
az container exec --resource-group myResourceGroup --name mycontainer --exec-command "/bin/bash"

# Delete container
az container delete --resource-group myResourceGroup --name mycontainer --yes

# Restart container
az container restart --resource-group myResourceGroup --name mycontainer
```

## Azure Kubernetes Service (AKS)

### Cluster Management
```bash
# List AKS clusters
az aks list
az aks list --resource-group myResourceGroup

# Create AKS cluster
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 3 \
  --enable-addons monitoring \
  --generate-ssh-keys

# Get AKS credentials
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

# Show AKS cluster
az aks show --resource-group myResourceGroup --name myAKSCluster

# Scale AKS cluster
az aks scale --resource-group myResourceGroup --name myAKSCluster --node-count 5

# Upgrade AKS cluster
az aks upgrade --resource-group myResourceGroup --name myAKSCluster --kubernetes-version 1.28.0

# Delete AKS cluster
az aks delete --resource-group myResourceGroup --name myAKSCluster --yes --no-wait
```

### AKS Operations
```bash
# Get available Kubernetes versions
az aks get-versions --location eastus

# Get node pool information
az aks nodepool list --resource-group myResourceGroup --cluster-name myAKSCluster

# Add node pool
az aks nodepool add \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name mynodepool \
  --node-count 2

# Start/Stop AKS cluster
az aks start --resource-group myResourceGroup --name myAKSCluster
az aks stop --resource-group myResourceGroup --name myAKSCluster
```

## Azure SQL Database

### SQL Server Management
```bash
# List SQL servers
az sql server list

# Create SQL server
az sql server create \
  --resource-group myResourceGroup \
  --name myserver \
  --location eastus \
  --admin-user myadmin \
  --admin-password MyPassword123!

# Show SQL server
az sql server show --resource-group myResourceGroup --name myserver

# Delete SQL server
az sql server delete --resource-group myResourceGroup --name myserver --yes

# Create firewall rule
az sql server firewall-rule create \
  --resource-group myResourceGroup \
  --server myserver \
  --name AllowMyIP \
  --start-ip-address 192.168.1.1 \
  --end-ip-address 192.168.1.1
```

### SQL Database Management
```bash
# List databases
az sql db list --resource-group myResourceGroup --server myserver

# Create database
az sql db create \
  --resource-group myResourceGroup \
  --server myserver \
  --name mydatabase \
  --service-objective S0

# Show database
az sql db show --resource-group myResourceGroup --server myserver --name mydatabase

# Delete database
az sql db delete --resource-group myResourceGroup --server myserver --name mydatabase --yes

# Create database backup
az sql db export \
  --resource-group myResourceGroup \
  --server myserver \
  --name mydatabase \
  --storage-key-type StorageAccessKey \
  --storage-key mykey \
  --storage-uri https://mystorageaccount.blob.core.windows.net/backups/backup.bacpac \
  --admin-user myadmin \
  --admin-password MyPassword123!
```

## Key Vault

### Key Vault Management
```bash
# List key vaults
az keyvault list

# Create key vault
az keyvault create \
  --resource-group myResourceGroup \
  --name myKeyVault \
  --location eastus

# Show key vault
az keyvault show --name myKeyVault --resource-group myResourceGroup

# Delete key vault
az keyvault delete --name myKeyVault --resource-group myResourceGroup

# Purge deleted key vault
az keyvault purge --name myKeyVault
```

### Secrets Management
```bash
# Set secret
az keyvault secret set --vault-name myKeyVault --name mySecret --value "mySecretValue"

# Get secret
az keyvault secret show --vault-name myKeyVault --name mySecret
az keyvault secret show --vault-name myKeyVault --name mySecret --query value -o tsv

# List secrets
az keyvault secret list --vault-name myKeyVault

# Delete secret
az keyvault secret delete --vault-name myKeyVault --name mySecret

# Backup secret
az keyvault secret backup --vault-name myKeyVault --name mySecret --file secret-backup.json

# Restore secret
az keyvault secret restore --vault-name myKeyVault --file secret-backup.json
```

### Keys and Certificates
```bash
# Create key
az keyvault key create --vault-name myKeyVault --name myKey --protection software

# List keys
az keyvault key list --vault-name myKeyVault

# Create certificate
az keyvault certificate create \
  --vault-name myKeyVault \
  --name myCertificate \
  --policy "$(az keyvault certificate get-default-policy)"

# List certificates
az keyvault certificate list --vault-name myKeyVault
```

## Networking

### Virtual Networks
```bash
# List virtual networks
az network vnet list

# Create virtual network
az network vnet create \
  --resource-group myResourceGroup \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnet \
  --subnet-prefix 10.0.1.0/24

# Show virtual network
az network vnet show --resource-group myResourceGroup --name myVNet

# Delete virtual network
az network vnet delete --resource-group myResourceGroup --name myVNet

# Create subnet
az network vnet subnet create \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet2 \
  --address-prefix 10.0.2.0/24
```

### Network Security Groups
```bash
# List NSGs
az network nsg list

# Create NSG
az network nsg create --resource-group myResourceGroup --name myNSG

# Create NSG rule
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name AllowSSH \
  --protocol tcp \
  --priority 1000 \
  --destination-port-range 22 \
  --access allow

# List NSG rules
az network nsg rule list --resource-group myResourceGroup --nsg-name myNSG

# Associate NSG with subnet
az network vnet subnet update \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet \
  --network-security-group myNSG
```

### Public IP Addresses
```bash
# List public IPs
az network public-ip list

# Create public IP
az network public-ip create \
  --resource-group myResourceGroup \
  --name myPublicIP \
  --allocation-method Static

# Show public IP
az network public-ip show --resource-group myResourceGroup --name myPublicIP

# Delete public IP
az network public-ip delete --resource-group myResourceGroup --name myPublicIP
```

## Load Balancer

### Load Balancer Management
```bash
# List load balancers
az network lb list

# Create load balancer
az network lb create \
  --resource-group myResourceGroup \
  --name myLoadBalancer \
  --public-ip-address myPublicIP \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool

# Show load balancer
az network lb show --resource-group myResourceGroup --name myLoadBalancer

# Create health probe
az network lb probe create \
  --resource-group myResourceGroup \
  --lb-name myLoadBalancer \
  --name myHealthProbe \
  --protocol tcp \
  --port 80

# Create load balancing rule
az network lb rule create \
  --resource-group myResourceGroup \
  --lb-name myLoadBalancer \
  --name myHTTPRule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool \
  --probe-name myHealthProbe
```

## Monitor & Logging

### Activity Logs
```bash
# Get activity logs
az monitor activity-log list
az monitor activity-log list --resource-group myResourceGroup

# Get activity logs for specific resource
az monitor activity-log list --resource-id "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"

# Get activity logs with filter
az monitor activity-log list --start-time 2023-01-01T00:00:00Z --end-time 2023-01-02T00:00:00Z
```

### Metrics
```bash
# List available metrics
az monitor metrics list-definitions --resource "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"

# Get metrics
az monitor metrics list \
  --resource "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM" \
  --metric "Percentage CPU" \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z
```

### Log Analytics
```bash
# List Log Analytics workspaces
az monitor log-analytics workspace list

# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group myResourceGroup \
  --workspace-name myWorkspace

# Query logs
az monitor log-analytics query \
  --workspace myWorkspace \
  --analytics-query "Heartbeat | take 10"
```

## Useful Options & Formatting

### Output Formats
```bash
# Table output (default for many commands)
az vm list --output table

# JSON output
az vm list --output json

# YAML output
az vm list --output yaml

# TSV (Tab-separated values)
az vm list --output tsv

# No output (useful for scripts)
az vm start --name myVM --resource-group myResourceGroup --output none
```

### JMESPath Querying
```bash
# Query specific fields
az vm list --query "[].{Name:name, ResourceGroup:resourceGroup, Location:location}"

# Filter results
az vm list --query "[?powerState=='VM running']"

# Get single value
az vm show --name myVM --resource-group myResourceGroup --query "vmId" --output tsv

# Complex queries
az vm list --query "[?contains(name, 'web')].{Name:name, Size:hardwareProfile.vmSize}"
```

### Batch Operations
```bash
# Use --no-wait for async operations
az vm start --name myVM --resource-group myResourceGroup --no-wait

# Use --yes to skip confirmations
az group delete --name myResourceGroup --yes

# Use --force for forced operations
az vm delete --name myVM --resource-group myResourceGroup --force
```

## Environment Variables

### Azure CLI Configuration
```bash
# Subscription
export AZURE_DEFAULTS_SUBSCRIPTION="subscription-id"

# Default resource group and location
export AZURE_DEFAULTS_GROUP="myResourceGroup"
export AZURE_DEFAULTS_LOCATION="eastus"

# Output format
export AZURE_CORE_OUTPUT="table"

# Disable telemetry
export AZURE_CORE_COLLECT_TELEMETRY="false"

# Authentication
export AZURE_CLIENT_ID="client-id"
export AZURE_CLIENT_SECRET="client-secret"
export AZURE_TENANT_ID="tenant-id"
```

## Troubleshooting & Debug

### Debug Options
```bash
# Enable debug output
az vm list --debug

# Verbose output
az vm list --verbose

# Check Azure CLI version
az version

# Update Azure CLI
az upgrade

# Check configuration
az config list
az account show

# Test connectivity
az rest --method get --url https://management.azure.com/subscriptions?api-version=2020-01-01
```

### Common Issues
```bash
# Clear token cache
az account clear
az login

# Check resource providers
az provider list --query "[?registrationState=='Registered']"

# Register resource provider
az provider register --namespace Microsoft.Compute

# Check quotas
az vm list-usage --location eastus

# Validate ARM templates
az deployment group validate --resource-group myResourceGroup --template-file template.json
```

## Tips & Best Practices

### Performance
1. **Use --no-wait** for long-running operations when possible
2. **Leverage JMESPath queries** to filter output client-side
3. **Use batch operations** instead of loops when available
4. **Cache authentication** by staying logged in
5. **Use --output tsv** for script-friendly output

### Security
1. **Use service principals** for automation
2. **Leverage managed identities** when running in Azure
3. **Store secrets in Key Vault**, not in scripts
4. **Use least privilege principle** for RBAC
5. **Regularly rotate credentials**

### Automation
1. **Set default values** to reduce command verbosity
2. **Use environment variables** for common parameters
3. **Implement proper error handling** in scripts
4. **Use ARM templates or Bicep** for infrastructure as code
5. **Tag resources** for better organization and cost tracking

### Useful Aliases
```bash
# Add to ~/.bashrc or ~/.zshrc
alias azwho='az account show --query user.name -o tsv'
alias azrg='az group list --output table'
alias azvm='az vm list --output table'
alias azsub='az account list --output table'
alias azloc='az account list-locations --output table'
```

### Script Examples
```bash
#!/bin/bash
# Create complete web app environment
RG_NAME="myWebAppRG"
APP_NAME="mywebapp$(date +%s)"
LOCATION="eastus"

# Create resource group
az group create --name $RG_NAME --location $LOCATION

# Create app service plan
az appservice plan create --resource-group $RG_NAME --name "${APP_NAME}-plan" --sku B1

# Create web app
az webapp create --resource-group $RG_NAME --plan "${APP_NAME}-plan" --name $APP_NAME --runtime "PYTHON|3.9"

# Output URL
echo "Web app URL: https://${APP_NAME}.azurewebsites.net"
```

