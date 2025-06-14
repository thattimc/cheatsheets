# AWS CLI Cheatsheet

## Installation & Setup

### Install AWS CLI
```bash
# macOS
brew install awscli

# Linux (Amazon Linux 2)
sudo yum install awscli

# Ubuntu/Debian
sudo apt install awscli

# pip install
pip install awscli

# Install AWS CLI v2 (recommended)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Configuration
```bash
# Configure AWS CLI (interactive)
aws configure

# Configure with specific profile
aws configure --profile myprofile

# Set individual configuration values
aws configure set aws_access_key_id AKIAIOSFODNN7EXAMPLE
aws configure set aws_secret_access_key wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
aws configure set region us-west-2
aws configure set output json

# List configuration
aws configure list
aws configure list --profile myprofile

# Get configuration values
aws configure get region
aws configure get aws_access_key_id --profile myprofile
```

### Profiles & Credentials
```bash
# List profiles
aws configure list-profiles

# Use specific profile
aws s3 ls --profile myprofile

# Set default profile via environment
export AWS_PROFILE=myprofile

# Set credentials via environment
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_DEFAULT_REGION=us-west-2
```

## EC2 (Elastic Compute Cloud)

### Instances
```bash
# List instances
aws ec2 describe-instances
aws ec2 describe-instances --instance-ids i-1234567890abcdef0

# List instances with specific filters
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
aws ec2 describe-instances --filters "Name=tag:Name,Values=MyServer"

# Start/Stop/Terminate instances
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Launch instance
aws ec2 run-instances \
    --image-id ami-12345678 \
    --count 1 \
    --instance-type t2.micro \
    --key-name MyKeyPair \
    --security-group-ids sg-903004f8 \
    --subnet-id subnet-6e7f829e

# Reboot instance
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0
```

### AMIs (Amazon Machine Images)
```bash
# List AMIs
aws ec2 describe-images --owners self
aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-*"

# Create AMI from instance
aws ec2 create-image \
    --instance-id i-1234567890abcdef0 \
    --name "My server" \
    --description "An AMI for my server"

# Delete AMI
aws ec2 deregister-image --image-id ami-12345678
```

### Security Groups
```bash
# List security groups
aws ec2 describe-security-groups
aws ec2 describe-security-groups --group-ids sg-903004f8

# Create security group
aws ec2 create-security-group \
    --group-name my-sg \
    --description "My security group"

# Add rules to security group
aws ec2 authorize-security-group-ingress \
    --group-id sg-903004f8 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# Remove rules from security group
aws ec2 revoke-security-group-ingress \
    --group-id sg-903004f8 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

### Key Pairs
```bash
# List key pairs
aws ec2 describe-key-pairs

# Create key pair
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem

# Delete key pair
aws ec2 delete-key-pair --key-name MyKeyPair
```

## S3 (Simple Storage Service)

### Buckets
```bash
# List buckets
aws s3 ls

# Create bucket
aws s3 mb s3://my-bucket-name
aws s3 mb s3://my-bucket-name --region us-west-2

# Delete bucket (must be empty)
aws s3 rb s3://my-bucket-name
aws s3 rb s3://my-bucket-name --force  # Delete with contents

# List bucket contents
aws s3 ls s3://my-bucket-name
aws s3 ls s3://my-bucket-name --recursive
aws s3 ls s3://my-bucket-name/path/ --human-readable --summarize
```

### Objects
```bash
# Upload file
aws s3 cp file.txt s3://my-bucket-name/
aws s3 cp file.txt s3://my-bucket-name/path/file.txt

# Download file
aws s3 cp s3://my-bucket-name/file.txt ./
aws s3 cp s3://my-bucket-name/file.txt ./downloaded-file.txt

# Sync directories
aws s3 sync ./local-folder s3://my-bucket-name/remote-folder
aws s3 sync s3://my-bucket-name/remote-folder ./local-folder

# Move files
aws s3 mv file.txt s3://my-bucket-name/
aws s3 mv s3://my-bucket-name/old-file.txt s3://my-bucket-name/new-file.txt

# Delete file
aws s3 rm s3://my-bucket-name/file.txt
aws s3 rm s3://my-bucket-name/path/ --recursive
```

### S3 Advanced Operations
```bash
# Set bucket versioning
aws s3api put-bucket-versioning \
    --bucket my-bucket-name \
    --versioning-configuration Status=Enabled

# Set bucket policy
aws s3api put-bucket-policy \
    --bucket my-bucket-name \
    --policy file://policy.json

# Get bucket location
aws s3api get-bucket-location --bucket my-bucket-name

# Set object ACL
aws s3api put-object-acl \
    --bucket my-bucket-name \
    --key file.txt \
    --acl public-read

# Generate presigned URL
aws s3 presign s3://my-bucket-name/file.txt --expires-in 3600
```

## IAM (Identity and Access Management)

### Users
```bash
# List users
aws iam list-users

# Create user
aws iam create-user --user-name myuser

# Delete user
aws iam delete-user --user-name myuser

# Get user details
aws iam get-user --user-name myuser

# Create access key for user
aws iam create-access-key --user-name myuser

# Delete access key
aws iam delete-access-key --user-name myuser --access-key-id AKIAIOSFODNN7EXAMPLE
```

### Groups
```bash
# List groups
aws iam list-groups

# Create group
aws iam create-group --group-name mygroup

# Add user to group
aws iam add-user-to-group --user-name myuser --group-name mygroup

# Remove user from group
aws iam remove-user-from-group --user-name myuser --group-name mygroup

# List users in group
aws iam get-group --group-name mygroup
```

### Policies
```bash
# List policies
aws iam list-policies
aws iam list-policies --scope Local  # Customer managed policies

# Create policy
aws iam create-policy \
    --policy-name MyPolicy \
    --policy-document file://policy.json

# Attach policy to user
aws iam attach-user-policy \
    --user-name myuser \
    --policy-arn arn:aws:iam::123456789012:policy/MyPolicy

# Detach policy from user
aws iam detach-user-policy \
    --user-name myuser \
    --policy-arn arn:aws:iam::123456789012:policy/MyPolicy

# List policies attached to user
aws iam list-attached-user-policies --user-name myuser
```

### Roles
```bash
# List roles
aws iam list-roles

# Create role
aws iam create-role \
    --role-name MyRole \
    --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
    --role-name MyRole \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Assume role
aws sts assume-role \
    --role-arn arn:aws:iam::123456789012:role/MyRole \
    --role-session-name MySession
```

## RDS (Relational Database Service)

### DB Instances
```bash
# List DB instances
aws rds describe-db-instances

# Create DB instance
aws rds create-db-instance \
    --db-instance-identifier mydb \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --master-username admin \
    --master-user-password mypassword \
    --allocated-storage 20

# Delete DB instance
aws rds delete-db-instance \
    --db-instance-identifier mydb \
    --skip-final-snapshot

# Start/Stop DB instance
aws rds start-db-instance --db-instance-identifier mydb
aws rds stop-db-instance --db-instance-identifier mydb

# Modify DB instance
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --db-instance-class db.t3.small \
    --apply-immediately
```

### Snapshots
```bash
# List snapshots
aws rds describe-db-snapshots
aws rds describe-db-snapshots --db-instance-identifier mydb

# Create snapshot
aws rds create-db-snapshot \
    --db-instance-identifier mydb \
    --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
    --db-instance-identifier mydb-restored \
    --db-snapshot-identifier mydb-snapshot-20231201

# Delete snapshot
aws rds delete-db-snapshot --db-snapshot-identifier mydb-snapshot-20231201
```

## Lambda

### Functions
```bash
# List functions
aws lambda list-functions

# Create function
aws lambda create-function \
    --function-name my-function \
    --runtime python3.9 \
    --role arn:aws:iam::123456789012:role/lambda-role \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip

# Update function code
aws lambda update-function-code \
    --function-name my-function \
    --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
    --function-name my-function \
    --payload '{"key1":"value1"}' \
    response.json

# Delete function
aws lambda delete-function --function-name my-function

# Get function configuration
aws lambda get-function-configuration --function-name my-function
```

### Function Configuration
```bash
# Update function configuration
aws lambda update-function-configuration \
    --function-name my-function \
    --timeout 30 \
    --memory-size 256

# Set environment variables
aws lambda update-function-configuration \
    --function-name my-function \
    --environment Variables='{"KEY1":"VALUE1","KEY2":"VALUE2"}'

# Add tags
aws lambda tag-resource \
    --resource arn:aws:lambda:us-west-2:123456789012:function:my-function \
    --tags Environment=Production,Team=Backend
```

## CloudFormation

### Stacks
```bash
# List stacks
aws cloudformation list-stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

# Create stack
aws cloudformation create-stack \
    --stack-name my-stack \
    --template-body file://template.yaml \
    --parameters ParameterKey=KeyName,ParameterValue=MyKeyPair

# Update stack
aws cloudformation update-stack \
    --stack-name my-stack \
    --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack

# Get stack events
aws cloudformation describe-stack-events --stack-name my-stack

# Get stack resources
aws cloudformation describe-stack-resources --stack-name my-stack
```

### Stack Operations
```bash
# Validate template
aws cloudformation validate-template --template-body file://template.yaml

# Estimate template cost
aws cloudformation estimate-template-cost --template-body file://template.yaml

# Get template
aws cloudformation get-template --stack-name my-stack

# Detect stack drift
aws cloudformation detect-stack-drift --stack-name my-stack
```

## CloudWatch

### Logs
```bash
# List log groups
aws logs describe-log-groups

# Create log group
aws logs create-log-group --log-group-name my-log-group

# Delete log group
aws logs delete-log-group --log-group-name my-log-group

# List log streams
aws logs describe-log-streams --log-group-name my-log-group

# Get log events
aws logs get-log-events \
    --log-group-name my-log-group \
    --log-stream-name my-log-stream

# Filter log events
aws logs filter-log-events \
    --log-group-name my-log-group \
    --filter-pattern "ERROR"
```

### Metrics
```bash
# List metrics
aws cloudwatch list-metrics
aws cloudwatch list-metrics --namespace AWS/EC2

# Get metric statistics
aws cloudwatch get-metric-statistics \
    --namespace AWS/EC2 \
    --metric-name CPUUtilization \
    --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
    --statistics Average \
    --start-time 2023-01-01T00:00:00Z \
    --end-time 2023-01-01T23:59:59Z \
    --period 3600

# Put metric data
aws cloudwatch put-metric-data \
    --namespace MyApp \
    --metric-data MetricName=MyMetric,Value=123,Unit=Count
```

## VPC (Virtual Private Cloud)

### VPC Management
```bash
# List VPCs
aws ec2 describe-vpcs

# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Delete VPC
aws ec2 delete-vpc --vpc-id vpc-12345678

# List subnets
aws ec2 describe-subnets
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-12345678"

# Create subnet
aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-west-2a

# Delete subnet
aws ec2 delete-subnet --subnet-id subnet-12345678
```

### Internet Gateway
```bash
# List internet gateways
aws ec2 describe-internet-gateways

# Create internet gateway
aws ec2 create-internet-gateway

# Attach internet gateway to VPC
aws ec2 attach-internet-gateway \
    --internet-gateway-id igw-12345678 \
    --vpc-id vpc-12345678

# Detach internet gateway from VPC
aws ec2 detach-internet-gateway \
    --internet-gateway-id igw-12345678 \
    --vpc-id vpc-12345678
```

## Route 53

### Hosted Zones
```bash
# List hosted zones
aws route53 list-hosted-zones

# Create hosted zone
aws route53 create-hosted-zone \
    --name example.com \
    --caller-reference $(date +%s)

# Delete hosted zone
aws route53 delete-hosted-zone --id Z123456789

# List resource record sets
aws route53 list-resource-record-sets --hosted-zone-id Z123456789
```

### DNS Records
```bash
# Create DNS record
aws route53 change-resource-record-sets \
    --hosted-zone-id Z123456789 \
    --change-batch file://change-batch.json

# Example change-batch.json for A record:
# {
#   "Changes": [{
#     "Action": "CREATE",
#     "ResourceRecordSet": {
#       "Name": "www.example.com",
#       "Type": "A",
#       "TTL": 300,
#       "ResourceRecords": [{
#         "Value": "192.0.2.1"
#       }]
#     }
#   }]
# }
```

## Useful Options & Formatting

### Output Formats
```bash
# JSON output (default)
aws s3 ls --output json

# Table output
aws s3 ls --output table

# Text output
aws s3 ls --output text

# YAML output
aws s3 ls --output yaml
```

### Filtering & Querying
```bash
# JMESPath query
aws ec2 describe-instances --query 'Reservations[*].Instances[*].InstanceId'
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name]'

# Filter by tags
aws ec2 describe-instances --query 'Reservations[*].Instances[?Tags[?Key==`Name`]|[0].Value==`MyServer`]'

# Get specific fields
aws s3api list-objects --bucket my-bucket --query 'Contents[].{Key: Key, Size: Size}'
```

### Pagination
```bash
# Get all results (handles pagination automatically)
aws s3api list-objects --bucket my-bucket --max-items 1000

# Manual pagination
aws s3api list-objects --bucket my-bucket --page-size 100

# Get next page using NextToken
aws s3api list-objects --bucket my-bucket --starting-token <token>
```

## Environment Variables

### AWS CLI Configuration
```bash
# Credentials
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_SESSION_TOKEN=your_session_token  # For temporary credentials

# Configuration
export AWS_DEFAULT_REGION=us-west-2
export AWS_DEFAULT_OUTPUT=json
export AWS_PROFILE=myprofile

# Other options
export AWS_CONFIG_FILE=~/.aws/config
export AWS_SHARED_CREDENTIALS_FILE=~/.aws/credentials
export AWS_CA_BUNDLE=/path/to/ca-bundle.pem
export AWS_CLI_AUTO_PROMPT=on
```

## Debugging & Troubleshooting

### Debug Options
```bash
# Enable debug output
aws s3 ls --debug

# Dry run (for supported commands)
aws ec2 run-instances --dry-run --image-id ami-12345678 --instance-type t2.micro

# No verify SSL (not recommended for production)
aws s3 ls --no-verify-ssl

# Increase CLI read timeout
aws s3 cp large-file.zip s3://my-bucket/ --cli-read-timeout 0
```

### Common Troubleshooting
```bash
# Check AWS CLI version
aws --version

# Check configuration
aws configure list
aws sts get-caller-identity

# Check regions
aws ec2 describe-regions

# Validate JSON
aws iam create-policy --policy-name test --policy-document file://policy.json --dry-run
```

## Tips & Best Practices

### Performance
1. **Use appropriate output formats** - `text` for scripts, `table` for humans
2. **Leverage JMESPath queries** to filter data client-side
3. **Use pagination** for large result sets
4. **Enable CLI pager** for better readability: `export AWS_PAGER=""`

### Security
1. **Use IAM roles** instead of access keys when possible
2. **Rotate access keys regularly**
3. **Use least privilege principle** for IAM policies
4. **Never commit credentials** to version control
5. **Use AWS SSO** for centralized access management

### Automation
1. **Use profiles** for different environments
2. **Leverage AWS CLI configuration files**
3. **Use environment variables** in CI/CD pipelines
4. **Implement proper error handling** in scripts
5. **Use AWS CLI v2** for better performance and features

### Useful Aliases
```bash
# Add to ~/.bashrc or ~/.zshrc
alias awswho='aws sts get-caller-identity'
alias awsregion='aws configure get region'
alias awsprofile='aws configure list-profiles'
alias ec2ls='aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==\`Name\`]|[0].Value]" --output table'
alias s3ls='aws s3 ls --human-readable --summarize'
```

