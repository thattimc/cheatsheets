# Terraform Cheatsheet

Terraform is an Infrastructure as Code (IaC) tool that allows you to build, change, and version infrastructure safely and efficiently across multiple cloud providers.

## Installation

### macOS
```bash
# Install via Homebrew
brew install terraform

# Install specific version
brew install terraform@1.5

# Install via tfenv (version manager)
brew install tfenv
tfenv install 1.5.0
tfenv use 1.5.0
```

### Linux
```bash
# Download and install binary
wget https://releases.hashicorp.com/terraform/1.5.0/terraform_1.5.0_linux_amd64.zip
unzip terraform_1.5.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify installation
terraform version

# Install via package manager (Ubuntu/Debian)
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update && sudo apt install terraform
```

### Windows
```powershell
# Using Chocolatey
choco install terraform

# Using winget
winget install HashiCorp.Terraform
```

## Basic Commands

### Essential Commands
```bash
# Initialize Terraform configuration
terraform init

# Create execution plan
terraform plan

# Apply changes
terraform apply

# Destroy infrastructure
terraform destroy

# Format code
terraform fmt

# Validate configuration
terraform validate

# Show current state
terraform show

# List resources in state
terraform state list

# Get provider versions
terraform version
```

### Advanced Commands
```bash
# Plan with specific var file
terraform plan -var-file="prod.tfvars"

# Apply with auto-approve
terraform apply -auto-approve

# Apply specific target
terraform apply -target=aws_instance.example

# Import existing resource
terraform import aws_instance.example i-1234567890abcdef0

# Refresh state
terraform refresh

# Output values
terraform output
terraform output instance_ip

# Workspace commands
terraform workspace list
terraform workspace new prod
terraform workspace select prod

# State commands
terraform state show aws_instance.example
terraform state mv aws_instance.old aws_instance.new
terraform state rm aws_instance.example
```

## Configuration Syntax

### Basic Configuration Structure
```hcl
# Configure Terraform and providers
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Configure providers
provider "aws" {
  region = var.aws_region
}

# Define variables
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
  validation {
    condition = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "Instance type must be t3.micro, t3.small, or t3.medium."
  }
}

# Define local values
locals {
  common_tags = {
    Environment = "production"
    Project     = "my-project"
  }
}

# Create resources
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  tags = merge(local.common_tags, {
    Name = "web-server"
  })
}

# Data sources
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# Output values
output "instance_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.web.public_ip
}

output "instance_dns" {
  description = "Public DNS of the instance"
  value       = aws_instance.web.public_dns
  sensitive   = false
}
```

## Variables

### Variable Types
```hcl
# String variable
variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

# Number variable
variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}

# Boolean variable
variable "enable_monitoring" {
  description = "Enable monitoring"
  type        = bool
  default     = true
}

# List variable
variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b"]
}

# Map variable
variable "instance_types" {
  description = "Map of instance types by environment"
  type        = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.large"
  }
}

# Object variable
variable "database_config" {
  description = "Database configuration"
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    allocated_storage = number
  })
  default = {
    engine         = "mysql"
    engine_version = "8.0"
    instance_class = "db.t3.micro"
    allocated_storage = 20
  }
}
```

### Variable Files
```hcl
# terraform.tfvars
aws_region = "us-east-1"
instance_type = "t3.small"
availability_zones = ["us-east-1a", "us-east-1b"]

# prod.tfvars
aws_region = "us-west-2"
instance_type = "t3.large"
enable_monitoring = true
instance_types = {
  web = "t3.large"
  db  = "r5.xlarge"
}
```

## Data Sources

### Common Data Sources
```hcl
# AWS AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# AWS VPC
data "aws_vpc" "default" {
  default = true
}

# AWS subnets
data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

# AWS availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Current AWS caller identity
data "aws_caller_identity" "current" {}

# AWS region
data "aws_region" "current" {}

# Remote state
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "my-terraform-state"
    key    = "network/terraform.tfstate"
    region = "us-west-2"
  }
}
```

## Resources

### AWS Examples
```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "main-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "main-igw"
  }
}

# Subnet
resource "aws_subnet" "public" {
  count = length(var.availability_zones)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${count.index + 1}"
  }
}

# Security Group
resource "aws_security_group" "web" {
  name_prefix = "web-"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "web-sg"
  }
}

# Launch Template
resource "aws_launch_template" "web" {
  name_prefix   = "web-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = base64encode(templatefile("${path.module}/user-data.sh", {
    environment = var.environment
  }))
  
  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "web-instance"
    }
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name                = "web-asg"
  vpc_zone_identifier = aws_subnet.public[*].id
  target_group_arns   = [aws_lb_target_group.web.arn]
  health_check_type   = "ELB"
  
  min_size         = 1
  max_size         = 3
  desired_capacity = 2
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
  
  tag {
    key                 = "Name"
    value               = "web-asg"
    propagate_at_launch = false
  }
}
```

## Modules

### Creating a Module
```hcl
# modules/vpc/main.tf
variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
}

variable "name" {
  description = "Name for VPC"
  type        = string
}

resource "aws_vpc" "this" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = var.name
  }
}

resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id
  
  tags = {
    Name = "${var.name}-igw"
  }
}

output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.this.id
}

output "vpc_cidr_block" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.this.cidr_block
}
```

### Using a Module
```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  
  cidr_block = "10.0.0.0/16"
  name       = "production-vpc"
}

# Using module from Terraform Registry
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = true
  
  tags = {
    Environment = "production"
  }
}

# Reference module outputs
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  subnet_id     = module.vpc.public_subnets[0]
}
```

## State Management

### Remote State Backend
```hcl
# S3 Backend
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# Azure Backend
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# Google Cloud Backend
terraform {
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "terraform/state"
  }
}
```

### State Commands
```bash
# List resources in state
terraform state list

# Show resource in state
terraform state show aws_instance.example

# Move resource in state
terraform state mv aws_instance.old aws_instance.new

# Remove resource from state
terraform state rm aws_instance.example

# Import existing resource
terraform import aws_instance.example i-1234567890abcdef0

# Pull remote state
terraform state pull

# Push local state to remote
terraform state push terraform.tfstate
```

## Functions

### Built-in Functions
```hcl
# String functions
locals {
  upper_env     = upper(var.environment)                    # "PROD"
  lower_name    = lower(var.name)                          # "my-app"
  title_case    = title(var.environment)                   # "Prod"
  join_list     = join("-", ["web", "server", "01"])       # "web-server-01"
  split_string  = split("-", "web-server-01")              # ["web", "server", "01"]
  substr_name   = substr(var.name, 0, 3)                   # "web"
  replace_text  = replace(var.name, "_", "-")              # Replace underscores
}

# Numeric functions
locals {
  max_value = max(5, 12, 9)                               # 12
  min_value = min(5, 12, 9)                               # 5
  abs_value = abs(-10)                                    # 10
  ceil_val  = ceil(5.1)                                   # 6
  floor_val = floor(5.9)                                  # 5
}

# Collection functions
locals {
  list_length   = length(["a", "b", "c"])                 # 3
  map_keys      = keys({a = 1, b = 2})                    # ["a", "b"]
  map_values    = values({a = 1, b = 2})                  # [1, 2]
  list_element  = element(["a", "b", "c"], 1)             # "b"
  list_index    = index(["a", "b", "c"], "b")             # 1
  list_contains = contains(["a", "b", "c"], "b")          # true
  list_concat   = concat(["a", "b"], ["c", "d"])          # ["a", "b", "c", "d"]
  list_distinct = distinct(["a", "b", "a", "c"])          # ["a", "b", "c"]
  list_sort     = sort(["c", "a", "b"])                   # ["a", "b", "c"]
  list_reverse  = reverse(["a", "b", "c"])                # ["c", "b", "a"]
}

# Map functions
locals {
  map_merge = merge(
    {environment = "prod"},
    {project = "my-app"}
  )  # {environment = "prod", project = "my-app"}
  
  map_lookup = lookup({
    dev  = "t3.micro"
    prod = "t3.large"
  }, var.environment, "t3.small")  # Default to t3.small if not found
}

# Date and time functions
locals {
  current_time = timestamp()                              # "2023-05-15T10:30:00Z"
  formatted_time = formatdate("YYYY-MM-DD", timestamp())   # "2023-05-15"
}

# Encoding functions
locals {
  base64_encoded = base64encode("hello world")            # "aGVsbG8gd29ybGQ="
  base64_decoded = base64decode("aGVsbG8gd29ybGQ=")        # "hello world"
  url_encoded    = urlencode("hello world")               # "hello%20world"
  json_encoded   = jsonencode({name = "John", age = 30})   # '{"age":30,"name":"John"}'
  yaml_encoded   = yamlencode({name = "John", age = 30})   # "age: 30\nname: John\n"
}

# File functions
locals {
  file_content    = file("${path.module}/config.txt")     # Read file content
  template_file   = templatefile("${path.module}/config.tpl", {
    environment = var.environment
  })
}
```

## Conditionals and Loops

### Conditional Expressions
```hcl
# Ternary operator
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
  
  tags = {
    Name = var.environment == "prod" ? "prod-server" : "dev-server"
  }
}

# Complex conditionals
locals {
  instance_type = (
    var.environment == "prod" ? "t3.large" :
    var.environment == "staging" ? "t3.medium" :
    "t3.micro"
  )
}
```

### Count
```hcl
# Create multiple instances
resource "aws_instance" "web" {
  count = var.instance_count
  
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  
  tags = {
    Name = "web-${count.index + 1}"
  }
}

# Conditional resource creation
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0
  
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
}
```

### for_each
```hcl
# Create resources from a map
variable "users" {
  type = map(object({
    role = string
  }))
  default = {
    alice = { role = "admin" }
    bob   = { role = "user" }
  }
}

resource "aws_iam_user" "users" {
  for_each = var.users
  
  name = each.key
  
  tags = {
    Role = each.value.role
  }
}

# Create resources from a set
variable "environments" {
  type    = set(string)
  default = ["dev", "staging", "prod"]
}

resource "aws_s3_bucket" "env_buckets" {
  for_each = var.environments
  
  bucket = "my-app-${each.value}"
  
  tags = {
    Environment = each.value
  }
}
```

### Dynamic Blocks
```hcl
resource "aws_security_group" "web" {
  name_prefix = "web-"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  default = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}
```

## Workspaces

### Workspace Commands
```bash
# List workspaces
terraform workspace list

# Create new workspace
terraform workspace new production

# Select workspace
terraform workspace select production

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete staging
```

### Using Workspaces in Configuration
```hcl
# Reference current workspace
locals {
  environment = terraform.workspace
  
  instance_type = {
    default = "t3.micro"
    prod    = "t3.large"
    staging = "t3.medium"
  }[terraform.workspace]
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = local.instance_type
  
  tags = {
    Name        = "web-${terraform.workspace}"
    Environment = terraform.workspace
  }
}
```

## Provisioners

### File Provisioner
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  key_name      = aws_key_pair.deployer.key_name
  
  provisioner "file" {
    source      = "conf/myapp.conf"
    destination = "/tmp/myapp.conf"
    
    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file(var.private_key_path)
      host        = self.public_ip
    }
  }
}
```

### Remote-exec Provisioner
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  provisioner "remote-exec" {
    inline = [
      "sudo yum update -y",
      "sudo yum install -y httpd",
      "sudo systemctl start httpd",
      "sudo systemctl enable httpd"
    ]
    
    connection {
      type        = "ssh"
      user        = "ec2-user"
      private_key = file(var.private_key_path)
      host        = self.public_ip
    }
  }
}
```

### Local-exec Provisioner
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> private_ips.txt"
  }
  
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Instance ${self.id} is being destroyed' >> destroy.log"
  }
}
```

## Best Practices

### Project Structure
```
project/
├── main.tf              # Main configuration
├── variables.tf         # Variable definitions
├── outputs.tf           # Output definitions
├── versions.tf          # Provider requirements
├── terraform.tfvars     # Variable values (don't commit secrets)
├── environments/
│   ├── dev.tfvars
│   ├── staging.tfvars
│   └── prod.tfvars
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── ec2/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── scripts/
    └── user-data.sh
```

### Naming Conventions
```hcl
# Use descriptive names
resource "aws_instance" "web_server" {  # Good
  # ...
}

resource "aws_instance" "i1" {  # Bad
  # ...
}

# Use consistent naming
resource "aws_security_group" "web_server_sg" {
  name = "web-server-sg"
}

resource "aws_instance" "web_server" {
  security_groups = [aws_security_group.web_server_sg.name]
}
```

### Security Best Practices
```hcl
# Don't hardcode secrets
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}

# Use data sources for sensitive values
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

# Mark outputs as sensitive
output "database_password" {
  value     = var.db_password
  sensitive = true
}

# Use least privilege IAM policies
data "aws_iam_policy_document" "s3_policy" {
  statement {
    effect = "Allow"
    actions = [
      "s3:GetObject",
      "s3:PutObject"
    ]
    resources = ["${aws_s3_bucket.app.arn}/*"]
  }
}
```

## Common Patterns

### Blue-Green Deployment
```hcl
variable "blue_instance_count" {
  description = "Number of blue instances"
  type        = number
  default     = 0
}

variable "green_instance_count" {
  description = "Number of green instances"
  type        = number
  default     = 2
}

resource "aws_instance" "blue" {
  count = var.blue_instance_count
  
  ami           = var.blue_ami
  instance_type = var.instance_type
  
  tags = {
    Name        = "blue-${count.index + 1}"
    Environment = "blue"
  }
}

resource "aws_instance" "green" {
  count = var.green_instance_count
  
  ami           = var.green_ami
  instance_type = var.instance_type
  
  tags = {
    Name        = "green-${count.index + 1}"
    Environment = "green"
  }
}
```

### Auto Scaling with Launch Template
```hcl
resource "aws_launch_template" "app" {
  name_prefix   = "app-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  
  vpc_security_group_ids = [aws_security_group.app.id]
  
  user_data = base64encode(templatefile("${path.module}/user-data.sh", {
    app_version = var.app_version
  }))
  
  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "app-instance"
    }
  }
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_autoscaling_group" "app" {
  name                = "app-asg"
  vpc_zone_identifier = var.subnet_ids
  target_group_arns   = [aws_lb_target_group.app.arn]
  health_check_type   = "ELB"
  
  min_size         = var.min_size
  max_size         = var.max_size
  desired_capacity = var.desired_capacity
  
  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }
  
  instance_refresh {
    strategy = "Rolling"
    preferences {
      min_healthy_percentage = 50
    }
  }
}
```

## Debugging and Troubleshooting

### Environment Variables
```bash
# Enable debug logging
export TF_LOG=DEBUG
export TF_LOG_PATH=./terraform.log

# Disable log output
export TF_LOG=

# Set specific log levels
export TF_LOG=TRACE  # Most verbose
export TF_LOG=DEBUG
export TF_LOG=INFO
export TF_LOG=WARN
export TF_LOG=ERROR

# Provider-specific logging
export TF_LOG_PROVIDER=DEBUG

# Core Terraform logging
export TF_LOG_CORE=DEBUG
```

### Common Commands for Debugging
```bash
# Validate configuration
terraform validate

# Check formatting
terraform fmt -check

# Plan with detailed output
terraform plan -out=tfplan
terraform show tfplan

# Graph visualization
terraform graph | dot -Tsvg > graph.svg

# Force unlock state
terraform force-unlock LOCK_ID

# Refresh state
terraform refresh

# Show providers
terraform providers

# Console for testing expressions
terraform console
```

## Useful Commands Reference

| Command | Description |
|---------|-------------|
| `terraform init` | Initialize working directory |
| `terraform plan` | Create execution plan |
| `terraform apply` | Apply changes |
| `terraform destroy` | Destroy infrastructure |
| `terraform fmt` | Format configuration files |
| `terraform validate` | Validate configuration |
| `terraform show` | Show current state |
| `terraform output` | Show output values |
| `terraform refresh` | Update state with real infrastructure |
| `terraform import` | Import existing resource into state |
| `terraform state list` | List resources in state |
| `terraform state show` | Show resource in state |
| `terraform workspace list` | List workspaces |
| `terraform providers` | Show provider requirements |
| `terraform version` | Show Terraform version |

## Environment Variables

```bash
# Terraform configuration
export TF_CLI_CONFIG_FILE="$HOME/.terraformrc"
export TF_DATA_DIR=".terraform"
export TF_INPUT=false
export TF_IN_AUTOMATION=true

# Logging
export TF_LOG=INFO
export TF_LOG_PATH="./terraform.log"

# Variables
export TF_VAR_region="us-west-2"
export TF_VAR_instance_type="t3.micro"

# Provider configuration
export AWS_REGION="us-west-2"
export AWS_PROFILE="default"
export GOOGLE_CREDENTIALS="path/to/credentials.json"
export ARM_SUBSCRIPTION_ID="subscription-id"
```

---

*For more detailed information, visit the [official Terraform documentation](https://www.terraform.io/docs)*

