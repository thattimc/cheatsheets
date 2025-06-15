# Google Cloud CLI (gcloud) Cheatsheet

## Installation & Setup

### Install Google Cloud CLI

```bash
# macOS
brew install google-cloud-sdk

# Linux (Debian/Ubuntu)
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] \
  https://packages.cloud.google.com/apt cloud-sdk main" \
  | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
  | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
sudo apt-get update && sudo apt-get install google-cloud-cli

# Linux (RHEL/CentOS)
sudo tee -a /etc/yum.repos.d/google-cloud-sdk.repo << EOM
[google-cloud-cli]
name=Google Cloud CLI
baseurl=https://packages.cloud.google.com/yum/repos/cloud-sdk-el8-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOM
sudo yum install google-cloud-cli

# Windows
# Download and run GoogleCloudSDKInstaller.exe from https://cloud.google.com/sdk/docs/install

# Docker
docker run -it google/cloud-sdk:latest gcloud version
```

### Authentication & Initialization

```bash
# Initialize gcloud (interactive setup)
gcloud init

# Authenticate with Google account
gcloud auth login

# Authenticate for application default credentials
gcloud auth application-default login

# Authenticate with service account
gcloud auth activate-service-account --key-file=path/to/key.json

# List authenticated accounts
gcloud auth list

# Set active account
gcloud config set account ACCOUNT

# Revoke authentication
gcloud auth revoke [ACCOUNT]
```

### Configuration

```bash
# List configurations
gcloud config configurations list

# Create new configuration
gcloud config configurations create my-config

# Activate configuration
gcloud config configurations activate my-config

# Set project
gcloud config set project PROJECT_ID

# Set default region/zone
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# List current configuration
gcloud config list

# Get specific configuration value
gcloud config get-value project
gcloud config get-value compute/zone

# Unset configuration
gcloud config unset compute/zone
```

## Projects

### Project Management

```bash
# List projects
gcloud projects list

# Create project
gcloud projects create PROJECT_ID --name="My Project"

# Get project details
gcloud projects describe PROJECT_ID

# Delete project
gcloud projects delete PROJECT_ID

# Set current project
gcloud config set project PROJECT_ID

# Get current project
gcloud config get-value project

# Link billing account to project
gcloud billing projects link PROJECT_ID --billing-account=BILLING_ACCOUNT_ID
```

## Compute Engine

### Instance Management

```bash
# List instances
gcloud compute instances list
gcloud compute instances list --zones=us-central1-a

# Create instance
gcloud compute instances create my-instance \
    --zone=us-central1-a \
    --machine-type=e2-medium \
    --image-family=ubuntu-2204-lts \
    --image-project=ubuntu-os-cloud

# Create instance with custom options
gcloud compute instances create my-instance \
    --zone=us-central1-a \
    --machine-type=e2-medium \
    --boot-disk-size=20GB \
    --boot-disk-type=pd-standard \
    --image-family=ubuntu-2204-lts \
    --image-project=ubuntu-os-cloud \
    --tags=http-server,https-server \
    --metadata=startup-script='#!/bin/bash\napt-get update'

# Start/Stop/Reset instance
gcloud compute instances start my-instance --zone=us-central1-a
gcloud compute instances stop my-instance --zone=us-central1-a
gcloud compute instances reset my-instance --zone=us-central1-a

# Delete instance
gcloud compute instances delete my-instance --zone=us-central1-a

# SSH into instance
gcloud compute ssh my-instance --zone=us-central1-a

# Copy files to/from instance
gcloud compute scp local-file my-instance:~/remote-file --zone=us-central1-a
gcloud compute scp my-instance:~/remote-file local-file --zone=us-central1-a
```

### Instance Information

```bash
# Describe instance
gcloud compute instances describe my-instance --zone=us-central1-a

# List machine types
gcloud compute machine-types list
gcloud compute machine-types list --zones=us-central1-a

# List images
gcloud compute images list
gcloud compute images list --project=ubuntu-os-cloud

# Get instance serial port output
gcloud compute instances get-serial-port-output my-instance --zone=us-central1-a

# List instance templates
gcloud compute instance-templates list
```

### Disks

```bash
# List disks
gcloud compute disks list

# Create disk
gcloud compute disks create my-disk \
    --size=100GB \
    --zone=us-central1-a \
    --type=pd-standard

# Attach disk to instance
gcloud compute instances attach-disk my-instance \
    --disk=my-disk \
    --zone=us-central1-a

# Detach disk from instance
gcloud compute instances detach-disk my-instance \
    --disk=my-disk \
    --zone=us-central1-a

# Create snapshot
gcloud compute disks snapshot my-disk \
    --snapshot-names=my-snapshot \
    --zone=us-central1-a

# Delete disk
gcloud compute disks delete my-disk --zone=us-central1-a
```

## Cloud Storage

### Bucket Management

```bash
# List buckets
gsutil ls
gsutil ls -b gs://bucket-name

# Create bucket
gsutil mb gs://my-bucket-name
gsutil mb -l us-central1 gs://my-bucket-name

# Delete bucket
gsutil rb gs://my-bucket-name
gsutil rb -f gs://my-bucket-name  # Force delete non-empty bucket

# Get bucket info
gsutil ls -L -b gs://my-bucket-name

# Set bucket versioning
gsutil versioning set on gs://my-bucket-name
gsutil versioning set off gs://my-bucket-name
```

### Object Management

```bash
# List objects
gsutil ls gs://my-bucket-name
gsutil ls -l gs://my-bucket-name/**
gsutil ls -r gs://my-bucket-name  # Recursive

# Upload files
gsutil cp local-file.txt gs://my-bucket-name/
gsutil cp -r local-directory gs://my-bucket-name/

# Download files
gsutil cp gs://my-bucket-name/file.txt ./
gsutil cp -r gs://my-bucket-name/directory ./

# Sync directories
gsutil rsync -r local-directory gs://my-bucket-name/remote-directory
gsutil rsync -r -d gs://my-bucket-name/remote-directory local-directory

# Move/rename objects
gsutil mv gs://my-bucket-name/old-name.txt gs://my-bucket-name/new-name.txt

# Delete objects
gsutil rm gs://my-bucket-name/file.txt
gsutil rm -r gs://my-bucket-name/directory

# Set object ACL
gsutil acl ch -u AllUsers:R gs://my-bucket-name/file.txt
gsutil acl ch -u user@example.com:O gs://my-bucket-name/file.txt

# Generate signed URL
gsutil signurl -d 1h path/to/service-account-key.json gs://my-bucket-name/file.txt
```

## Google Kubernetes Engine (GKE)

### Cluster Management

```bash
# List clusters
gcloud container clusters list

# Create cluster
gcloud container clusters create my-cluster \
    --zone=us-central1-a \
    --num-nodes=3 \
    --machine-type=e2-medium

# Create cluster with additional options
gcloud container clusters create my-cluster \
    --zone=us-central1-a \
    --num-nodes=3 \
    --machine-type=e2-medium \
    --disk-size=50GB \
    --enable-autoscaling \
    --min-nodes=1 \
    --max-nodes=5 \
    --enable-autorepair \
    --enable-autoupgrade

# Get cluster credentials
gcloud container clusters get-credentials my-cluster --zone=us-central1-a

# Resize cluster
gcloud container clusters resize my-cluster --num-nodes=5 --zone=us-central1-a

# Upgrade cluster
gcloud container clusters upgrade my-cluster --zone=us-central1-a

# Delete cluster
gcloud container clusters delete my-cluster --zone=us-central1-a
```

### Node Pool Management

```bash
# List node pools
gcloud container node-pools list --cluster=my-cluster --zone=us-central1-a

# Create node pool
gcloud container node-pools create my-node-pool \
    --cluster=my-cluster \
    --zone=us-central1-a \
    --num-nodes=2 \
    --machine-type=e2-standard-2

# Delete node pool
gcloud container node-pools delete my-node-pool \
    --cluster=my-cluster \
    --zone=us-central1-a
```

## App Engine

### Application Management

```bash
# Deploy application
gcloud app deploy
gcloud app deploy --version=v1 --promote

# Deploy specific service
gcloud app deploy service.yaml

# List versions
gcloud app versions list
gcloud app versions list --service=default

# Set traffic allocation
gcloud app services set-traffic default --splits=v1=.5,v2=.5

# Stop version
gcloud app versions stop v1 --service=default

# Delete version
gcloud app versions delete v1 --service=default

# Browse application
gcloud app browse
gcloud app browse --service=api --version=v1

# View logs
gcloud app logs tail -s default
gcloud app logs read -s default --limit=100
```

### Application Configuration

```bash
# Describe application
gcloud app describe

# List services
gcloud app services list

# List instances
gcloud app instances list

# SSH into instance
gcloud app instances ssh INSTANCE_ID --service=default --version=v1
```

## Cloud Functions

### Function Management

```bash
# List functions
gcloud functions list

# Deploy function
gcloud functions deploy my-function \
    --runtime=python39 \
    --trigger-http \
    --allow-unauthenticated \
    --source=.

# Deploy function with trigger
gcloud functions deploy my-function \
    --runtime=python39 \
    --trigger-bucket=my-bucket \
    --source=.

# Call function
gcloud functions call my-function --data='{"key":"value"}'

# Get function details
gcloud functions describe my-function

# View function logs
gcloud functions logs read my-function

# Delete function
gcloud functions delete my-function
```

## Cloud SQL

### SQL Instance Management

```bash
# List instances
gcloud sql instances list

# Create instance
gcloud sql instances create my-instance \
    --database-version=MYSQL_8_0 \
    --tier=db-f1-micro \
    --region=us-central1

# Create PostgreSQL instance
gcloud sql instances create my-pg-instance \
    --database-version=POSTGRES_13 \
    --tier=db-f1-micro \
    --region=us-central1

# Connect to instance
gcloud sql connect my-instance --user=root

# Start/Stop instance
gcloud sql instances patch my-instance --activation-policy=ALWAYS
gcloud sql instances patch my-instance --activation-policy=NEVER

# Delete instance
gcloud sql instances delete my-instance
```

### Database and User Management

```bash
# List databases
gcloud sql databases list --instance=my-instance

# Create database
gcloud sql databases create my-database --instance=my-instance

# Delete database
gcloud sql databases delete my-database --instance=my-instance

# List users
gcloud sql users list --instance=my-instance

# Create user
gcloud sql users create myuser --instance=my-instance --password=mypassword

# Set user password
gcloud sql users set-password myuser --instance=my-instance --password=newpassword

# Delete user
gcloud sql users delete myuser --instance=my-instance
```

### Backups and Export/Import

```bash
# List backups
gcloud sql backups list --instance=my-instance

# Create backup
gcloud sql backups create --instance=my-instance

# Restore from backup
gcloud sql backups restore BACKUP_ID --restore-instance=my-instance

# Export database
gcloud sql export sql my-instance gs://my-bucket/database-export.sql \
    --database=my-database

# Import database
gcloud sql import sql my-instance gs://my-bucket/database-export.sql \
    --database=my-database
```

## Identity and Access Management (IAM)

### IAM Policies

```bash
# Get IAM policy
gcloud projects get-iam-policy PROJECT_ID

# Add IAM binding
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member=user:user@example.com \
    --role=roles/editor

# Add service account binding
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member=serviceAccount:sa@PROJECT_ID.iam.gserviceaccount.com \
    --role=roles/storage.admin

# Remove IAM binding
gcloud projects remove-iam-policy-binding PROJECT_ID \
    --member=user:user@example.com \
    --role=roles/editor

# List IAM roles
gcloud iam roles list
gcloud iam roles list --project=PROJECT_ID

# Describe IAM role
gcloud iam roles describe roles/editor
```

### Service Accounts

```bash
# List service accounts
gcloud iam service-accounts list

# Create service account
gcloud iam service-accounts create my-service-account \
    --display-name="My Service Account" \
    --description="Service account for my application"

# Delete service account
gcloud iam service-accounts delete my-service-account@PROJECT_ID.iam.gserviceaccount.com

# Create and download key
gcloud iam service-accounts keys create key.json \
    --iam-account=my-service-account@PROJECT_ID.iam.gserviceaccount.com

# List service account keys
gcloud iam service-accounts keys list \
    --iam-account=my-service-account@PROJECT_ID.iam.gserviceaccount.com

# Delete service account key
gcloud iam service-accounts keys delete KEY_ID \
    --iam-account=my-service-account@PROJECT_ID.iam.gserviceaccount.com
```

## Networking

### VPC Networks

```bash
# List networks
gcloud compute networks list

# Create network
gcloud compute networks create my-network --subnet-mode=custom

# Create subnet
gcloud compute networks subnets create my-subnet \
    --network=my-network \
    --range=10.1.0.0/16 \
    --region=us-central1

# List subnets
gcloud compute networks subnets list

# Delete network
gcloud compute networks delete my-network
```

### Firewall Rules

```bash
# List firewall rules
gcloud compute firewall-rules list

# Create firewall rule
gcloud compute firewall-rules create allow-http \
    --allow=tcp:80 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=http-server

# Create firewall rule for SSH
gcloud compute firewall-rules create allow-ssh \
    --allow=tcp:22 \
    --source-ranges=0.0.0.0/0

# Delete firewall rule
gcloud compute firewall-rules delete allow-http

# Update firewall rule
gcloud compute firewall-rules update allow-http --allow=tcp:80,tcp:443
```

### Load Balancers

```bash
# Create HTTP load balancer
gcloud compute backend-services create my-backend-service \
    --protocol=HTTP \
    --global

# Create URL map
gcloud compute url-maps create my-url-map \
    --default-service=my-backend-service

# Create HTTP proxy
gcloud compute target-http-proxies create my-http-proxy \
    --url-map=my-url-map

# Create forwarding rule
gcloud compute forwarding-rules create my-forwarding-rule \
    --global \
    --target-http-proxy=my-http-proxy \
    --ports=80
```

## BigQuery

### Dataset Management

```bash
# List datasets
bq ls
bq ls PROJECT_ID:

# Create dataset
bq mk my_dataset
bq mk --location=US my_dataset

# Show dataset info
bq show my_dataset

# Delete dataset
bq rm -r my_dataset
```

### Table Management

```bash
# List tables
bq ls my_dataset

# Create table
bq mk my_dataset.my_table schema.json

# Load data from CSV
bq load --source_format=CSV my_dataset.my_table data.csv schema.json

# Load data from Cloud Storage
bq load my_dataset.my_table gs://my-bucket/data.csv

# Show table schema
bq show my_dataset.my_table

# Query table
bq query "SELECT * FROM my_dataset.my_table LIMIT 10"

# Export table to Cloud Storage
bq extract my_dataset.my_table gs://my-bucket/export.csv

# Delete table
bq rm my_dataset.my_table
```

### Query Operations

```bash
# Run query
bq query "SELECT COUNT(*) FROM my_dataset.my_table"

# Run query with parameters
bq query --use_legacy_sql=false \
    --parameter=name:STRING:John \
    "SELECT * FROM my_dataset.my_table WHERE name = @name"

# Save query results to table
bq query --destination_table=my_dataset.results \
    "SELECT * FROM my_dataset.my_table WHERE condition = true"

# Dry run query (estimate cost)
bq query --dry_run "SELECT * FROM my_dataset.my_table"
```

## Cloud Logging

### Log Management

```bash
# Read logs
gcloud logging read "resource.type=gce_instance"
gcloud logging read "resource.type=gce_instance" --limit=10

# Read logs with filter
gcloud logging read "resource.type=gce_instance AND severity=ERROR"

# Read logs for specific time range
gcloud logging read "resource.type=gce_instance" \
    --freshness=1h

# Tail logs
gcloud logging tail "resource.type=gce_instance"

# List log entries
gcloud logging entries list --filter="resource.type=gce_instance"
```

### Log Sinks

```bash
# List sinks
gcloud logging sinks list

# Create sink to Cloud Storage
gcloud logging sinks create my-sink \
    gs://my-bucket/logs \
    --log-filter="resource.type=gce_instance"

# Create sink to BigQuery
gcloud logging sinks create my-bq-sink \
    bigquery.googleapis.com/projects/PROJECT_ID/datasets/my_dataset \
    --log-filter="severity=ERROR"

# Delete sink
gcloud logging sinks delete my-sink
```

## Cloud Monitoring

### Metrics

```bash
# List metric descriptors
gcloud monitoring metrics list

# List time series
gcloud monitoring metrics list-time-series \
    --filter='metric.type="compute.googleapis.com/instance/cpu/utilization"'

# Create alerting policy
gcloud alpha monitoring policies create --policy-from-file=policy.yaml
```

## Secret Manager

### Secret Management

```bash
# Create secret
gcloud secrets create my-secret --data-file=secret.txt
echo "my-secret-value" | gcloud secrets create my-secret --data-file=-

# List secrets
gcloud secrets list

# Access secret
gcloud secrets versions access latest --secret=my-secret

# Add new version
echo "new-secret-value" | gcloud secrets versions add my-secret --data-file=-

# List versions
gcloud secrets versions list my-secret

# Delete secret
gcloud secrets delete my-secret
```

## Useful Options & Formatting

### Output Formats

```bash
# JSON output
gcloud compute instances list --format=json

# YAML output
gcloud compute instances list --format=yaml

# Table output with custom columns
gcloud compute instances list --format="table(name,machineType,status)"

# CSV output
gcloud compute instances list --format="csv(name,machineType,status)"

# Value output (single values)
gcloud config get-value project --format="value()"

# No headers in table
gcloud compute instances list --format="table[no-heading](name,status)"
```

### Filtering and Projection

```bash
# Filter results
gcloud compute instances list --filter="status:RUNNING"
gcloud compute instances list --filter="zone:us-central1-a"

# Sort results
gcloud compute instances list --sort-by=name
gcloud compute instances list --sort-by=~creationTimestamp  # Reverse sort

# Limit results
gcloud compute instances list --limit=10

# Custom projections
gcloud compute instances list --format="value(name,machineType)"
```

### Global Flags

```bash
# Specify project
gcloud compute instances list --project=my-project

# Quiet mode (no prompts)
gcloud compute instances delete my-instance --quiet

# Verbose output
gcloud compute instances create my-instance --verbosity=debug

# No user output
gcloud compute instances create my-instance --verbosity=none
```

## Environment Variables

### Environment Configuration

```bash
# Set default project
export GOOGLE_CLOUD_PROJECT=my-project
export GCLOUD_PROJECT=my-project

# Set default region/zone
export GOOGLE_CLOUD_REGION=us-central1
export GOOGLE_CLOUD_ZONE=us-central1-a

# Service account credentials
export GOOGLE_APPLICATION_CREDENTIALS=path/to/service-account-key.json

# Disable prompts
export CLOUDSDK_CORE_DISABLE_PROMPTS=1

# Set output format
export CLOUDSDK_CORE_FORMAT=json
```

## Troubleshooting & Debug

### Debug Options

```bash
# Check gcloud version
gcloud version

# Update gcloud
gcloud components update

# List installed components
gcloud components list

# Install additional components
gcloud components install kubectl
gcloud components install app-engine-python

# Check configuration
gcloud config list
gcloud info

# Test connectivity
gcloud auth application-default print-access-token

# Enable debug logging
gcloud compute instances list --log-http
```

### Common Issues

```bash
# Clear credentials cache
gcloud auth application-default revoke
gcloud auth revoke --all

# Re-authenticate
gcloud auth login
gcloud auth application-default login

# Check quotas
gcloud compute project-info describe --project=PROJECT_ID

# List enabled APIs
gcloud services list --enabled

# Enable API
gcloud services enable compute.googleapis.com
gcloud services enable storage.googleapis.com
```

## Tips & Best Practices

### Performance

1. **Use filters** to reduce data transferred and improve performance
2. **Set defaults** in configuration to avoid repeating parameters
3. **Use batch operations** when possible
4. **Leverage concurrent uploads** with gsutil -m flag
5. **Use appropriate machine types** for your workloads

### Security

1. **Use service accounts** for applications and automation
2. **Follow principle of least privilege** for IAM roles
3. **Enable audit logging** for compliance
4. **Rotate service account keys** regularly
5. **Use Secret Manager** for sensitive data

### Cost Optimization

1. **Use sustained use discounts** for long-running instances
2. **Implement preemptible instances** for fault-tolerant workloads
3. **Monitor usage** with Cloud Billing API
4. **Set up budget alerts** to avoid surprises
5. **Clean up unused resources** regularly

### Automation

1. **Use deployment manager** or Terraform for infrastructure as code
2. **Implement CI/CD pipelines** with Cloud Build
3. **Use Cloud Scheduler** for recurring tasks
4. **Leverage Cloud Functions** for event-driven automation
5. **Set up monitoring and alerting** for production workloads

### Useful Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc
alias gcl='gcloud compute instances list'
alias gcs='gcloud compute ssh'
alias gcp='gcloud config get-value project'
alias gcz='gcloud config get-value compute/zone'
alias glogin='gcloud auth login'
alias gswitch='gcloud config configurations activate'
```

### Script Examples

```bash
#!/bin/bash
# Create development environment
PROJECT_ID="my-dev-project"
ZONE="us-central1-a"
INSTANCE_NAME="dev-instance"

# Set project
gcloud config set project $PROJECT_ID

# Create instance
gcloud compute instances create $INSTANCE_NAME \
    --zone=$ZONE \
    --machine-type=e2-medium \
    --image-family=ubuntu-2204-lts \
    --image-project=ubuntu-os-cloud \
    --tags=http-server

# Create firewall rule
gcloud compute firewall-rules create allow-http-dev \
    --allow=tcp:80,tcp:8080 \
    --target-tags=http-server

echo "Development environment created!"
echo "SSH: gcloud compute ssh $INSTANCE_NAME --zone=$ZONE"
```

### Multi-line Commands

```bash
# Use backslashes for readability
gcloud compute instances create my-instance \
    --zone=us-central1-a \
    --machine-type=e2-medium \
    --boot-disk-size=20GB \
    --image-family=ubuntu-2204-lts \
    --image-project=ubuntu-os-cloud \
    --metadata=startup-script='#!/bin/bash
        apt-get update
        apt-get install -y nginx
        systemctl start nginx' \
    --tags=http-server
```

## Advanced Features

### Custom Configurations

```bash
# Create configuration for different environments
gcloud config configurations create development
gcloud config configurations create production

# Switch between configurations
gcloud config configurations activate development
gcloud config set project dev-project-123
gcloud config set compute/zone us-central1-a

gcloud config configurations activate production
gcloud config set project prod-project-456
gcloud config set compute/zone us-east1-a
```

### Batch Operations

```bash
# Create multiple instances
for i in {1..3}; do
    gcloud compute instances create "web-$i" \
        --zone=us-central1-a \
        --machine-type=e2-micro \
        --async
done

# Wait for operations to complete
gcloud compute operations wait OPERATION_NAME --zone=us-central1-a
```
