# OpenTofu Cheatsheet

OpenTofu is an open-source Infrastructure as Code (IaC) tool and the community-driven
fork of Terraform. It provides the same functionality as Terraform with additional features
and improvements.

## Installation

### macOS

```bash
# Install via Homebrew
brew install opentofu

# Install specific version
brew install opentofu@1.6

# Install via tfenv (supports OpenTofu)
brew install tfenv
tfenv install opentofu-1.6.0
tfenv use opentofu-1.6.0

# Verify installation
tofu version
```

### Linux

```bash
# Download and install binary
wget https://github.com/opentofu/opentofu/releases/download/v1.6.0/tofu_1.6.0_linux_amd64.zip
unzip tofu_1.6.0_linux_amd64.zip
sudo mv tofu /usr/local/bin/

# Verify installation
tofu version

# Install via package manager (Ubuntu/Debian)
curl -fsSL https://get.opentofu.org/install-opentofu.sh | sudo bash

# Install via Snap
sudo snap install opentofu --classic
```

### Windows

```powershell
# Using Chocolatey
choco install opentofu

# Using winget
winget install OpenTofu.Tofu

# Using Scoop
scoop install opentofu
```

## Basic Commands

### Essential Commands

```bash
# Initialize OpenTofu configuration
tofu init

# Create execution plan
tofu plan

# Apply changes
tofu apply

# Destroy infrastructure
tofu destroy

# Format code
tofu fmt

# Validate configuration
tofu validate

# Show current state
tofu show

# List resources in state
tofu state list

# Get provider versions
tofu version
```

### Advanced Commands

```bash
# Plan with specific var file
tofu plan -var-file="prod.tfvars"

# Apply with auto-approve
tofu apply -auto-approve

# Apply specific target
tofu apply -target=aws_instance.example

# Import existing resource
tofu import aws_instance.example i-1234567890abcdef0

# Refresh state
tofu refresh

# Output values
tofu output
tofu output instance_ip

# Workspace commands
tofu workspace list
tofu workspace new prod
tofu workspace select prod

# State commands
tofu state show aws_instance.example
tofu state mv aws_instance.old aws_instance.new
tofu state rm aws_instance.example
```

## Configuration Syntax

### Basic Configuration Structure

```hcl
# Configure OpenTofu and providers
terraform {
  required_version = ">= 1.6"
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
    ManagedBy   = "OpenTofu"
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

## OpenTofu-Specific Features

### State Encryption

```hcl
# OpenTofu supports state encryption
terraform {
  encryption {
    key_provider "pbkdf2" "my_passphrase" {
      passphrase = var.passphrase
    }
    
    method "aes_gcm" "my_method" {
      keys = key_provider.pbkdf2.my_passphrase
    }
    
    state {
      method = method.aes_gcm.my_method
    }
    
    plan {
      method = method.aes_gcm.my_method
    }
  }
}
```

### Provider-defined Functions

```hcl
# OpenTofu supports provider-defined functions
locals {
  # Using provider functions (when available)
  encoded_data = provider::aws::base64encode("hello world")
  
  # Traditional approach still works
  encoded_data_builtin = base64encode("hello world")
}
```

### Improved Testing

```hcl
# OpenTofu has enhanced testing capabilities
# test/main.tftest.hcl
run "test_instance_creation" {
  command = plan
  
  variables {
    instance_type = "t3.micro"
    aws_region    = "us-west-2"
  }
  
  assert {
    condition     = aws_instance.web.instance_type == "t3.micro"
    error_message = "Instance type should be t3.micro"
  }
}

run "test_instance_tags" {
  command = plan
  
  assert {
    condition     = aws_instance.web.tags["ManagedBy"] == "OpenTofu"
    error_message = "Instance should be tagged with ManagedBy = OpenTofu"
  }
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
    engine            = string
    engine_version    = string
    instance_class    = string
    allocated_storage = number
  })
  default = {
    engine            = "mysql"
    engine_version    = "8.0"
    instance_class    = "db.t3.micro"
    allocated_storage = 20
  }
}

# OpenTofu enhanced validation
variable "instance_size" {
  description = "Instance size with enhanced validation"
  type        = string
  default     = "small"
  
  validation {
    condition = can(regex("^(small|medium|large)$", var.instance_size))
    error_message = "Instance size must be small, medium, or large."
  }
  
  validation {
    condition = var.instance_size != "large" || var.environment == "prod"
    error_message = "Large instances are only allowed in production."
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

# OpenTofu supports .tofu.tfvars files
# production.tofu.tfvars
environment = "production"
instance_count = 3
enable_monitoring = true
```

## State Management

### Remote State Backend

```hcl
# S3 Backend with encryption
terraform {
  backend "s3" {
    bucket         = "my-opentofu-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "opentofu-locks"
  }
  
  # OpenTofu state encryption
  encryption {
    key_provider "pbkdf2" "my_key" {
      passphrase = var.state_passphrase
    }
    
    method "aes_gcm" "my_method" {
      keys = key_provider.pbkdf2.my_key
    }
    
    state {
      method = method.aes_gcm.my_method
    }
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
    bucket = "my-opentofu-state"
    prefix = "terraform/state"
  }
}
```

### State Commands

```bash
# List resources in state
tofu state list

# Show resource in state
tofu state show aws_instance.example

# Move resource in state
tofu state mv aws_instance.old aws_instance.new

# Remove resource from state
tofu state rm aws_instance.example

# Import existing resource
tofu import aws_instance.example i-1234567890abcdef0

# Pull remote state
tofu state pull

# Push local state to remote
tofu state push terraform.tfstate

# OpenTofu-specific: Encrypt existing state
tofu state encrypt
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

# OpenTofu enhanced functions
locals {
  # Enhanced string manipulation
  trimmed_string = trim("  hello world  ", " ")           # "hello world"
  padded_string  = strrev("hello")                         # "olleh"
  
  # Enhanced collection functions
  flatten_list = flatten([["a", "b"], ["c", "d"]])        # ["a", "b", "c", "d"]
  zip_lists    = zipmap(["a", "b"], [1, 2])               # {"a" = 1, "b" = 2}
}

# File functions
locals {
  file_content    = file("${path.module}/config.txt")     # Read file content
  template_file   = templatefile("${path.module}/config.tpl", {
    environment = var.environment
  })
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

variable "tags" {
  description = "Additional tags"
  type        = map(string)
  default     = {}
}

resource "aws_vpc" "this" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(
    {
      Name      = var.name
      ManagedBy = "OpenTofu"
    },
    var.tags
  )
}

resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id
  
  tags = merge(
    {
      Name      = "${var.name}-igw"
      ManagedBy = "OpenTofu"
    },
    var.tags
  )
}

output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.this.id
}

output "vpc_cidr_block" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.this.cidr_block
}

output "internet_gateway_id" {
  description = "ID of the Internet Gateway"
  value       = aws_internet_gateway.this.id
}
```

### Using a Module

```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  
  cidr_block = "10.0.0.0/16"
  name       = "production-vpc"
  
  tags = {
    Environment = "production"
    Team        = "infrastructure"
  }
}

# Using module from OpenTofu Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
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
    ManagedBy   = "OpenTofu"
  }
}

# Reference module outputs
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  subnet_id     = module.vpc.public_subnets[0]
  
  tags = {
    Name      = "web-server"
    ManagedBy = "OpenTofu"
  }
}
```

## Testing with OpenTofu

### Test Configuration

```hcl
# tests/vpc_test.tftest.hcl
variables {
  cidr_block = "10.0.0.0/16"
  name       = "test-vpc"
}

run "test_vpc_creation" {
  command = plan
  
  assert {
    condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR block should be 10.0.0.0/16"
  }
  
  assert {
    condition     = aws_vpc.main.enable_dns_hostnames == true
    error_message = "DNS hostnames should be enabled"
  }
}

run "test_internet_gateway" {
  command = plan
  
  assert {
    condition     = aws_internet_gateway.main.vpc_id == aws_vpc.main.id
    error_message = "Internet gateway should be attached to the VPC"
  }
}

run "test_tags" {
  command = plan
  
  assert {
    condition     = aws_vpc.main.tags["ManagedBy"] == "OpenTofu"
    error_message = "VPC should be tagged with ManagedBy = OpenTofu"
  }
}
```

### Running Tests

```bash
# Run all tests
tofu test

# Run specific test file
tofu test tests/vpc_test.tftest.hcl

# Run tests with verbose output
tofu test -verbose

# Run tests and generate report
tofu test -json > test_results.json
```

## OpenTofu vs Terraform Differences

### Key Differences

```hcl
# 1. State Encryption (OpenTofu only)
terraform {
  encryption {
    key_provider "pbkdf2" "my_key" {
      passphrase = var.passphrase
    }
    
    method "aes_gcm" "my_method" {
      keys = key_provider.pbkdf2.my_key
    }
    
    state {
      method = method.aes_gcm.my_method
    }
  }
}

# 2. Enhanced Testing Framework
# OpenTofu has built-in testing with .tftest.hcl files

# 3. Provider-defined Functions
# OpenTofu supports provider-defined functions
locals {
  # Using provider functions (when available)
  result = provider::aws::custom_function("input")
}

# 4. Enhanced Error Messages
# OpenTofu provides more detailed error messages and suggestions

# 5. Community Governance
# OpenTofu is governed by the Linux Foundation
```

### Migration from Terraform

```bash
# 1. Replace terraform commands with tofu
# terraform init -> tofu init
# terraform plan -> tofu plan
# terraform apply -> tofu apply

# 2. Update version constraints (optional)
terraform {
  required_version = ">= 1.6"  # OpenTofu version
}

# 3. No changes needed to existing .tf files
# OpenTofu is fully compatible with Terraform HCL

# 4. State file migration (if needed)
tofu init -migrate-state
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
│   │   ├── outputs.tf
│   │   └── versions.tf
│   └── ec2/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── versions.tf
├── tests/
│   ├── vpc_test.tftest.hcl
│   └── ec2_test.tftest.hcl
└── scripts/
    └── user-data.sh
```

### Security Best Practices

```hcl
# Use state encryption
terraform {
  encryption {
    key_provider "pbkdf2" "state_key" {
      passphrase = var.state_passphrase
    }
    
    method "aes_gcm" "state_method" {
      keys = key_provider.pbkdf2.state_key
    }
    
    state {
      method = method.aes_gcm.state_method
    }
    
    plan {
      method = method.aes_gcm.state_method
    }
  }
}

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

# Always tag resources with ManagedBy
locals {
  common_tags = {
    ManagedBy   = "OpenTofu"
    Environment = var.environment
    Project     = var.project_name
  }
}
```

## Debugging and Troubleshooting

### Environment Variables

```bash
# Enable debug logging
export TF_LOG=DEBUG
export TF_LOG_PATH=./opentofu.log

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

# Core OpenTofu logging
export TF_LOG_CORE=DEBUG

# OpenTofu-specific variables
export TOFU_LOG=DEBUG
export TOFU_LOG_PATH=./tofu.log
```

### Common Commands for Debugging

```bash
# Validate configuration
tofu validate

# Check formatting
tofu fmt -check

# Plan with detailed output
tofu plan -out=tfplan
tofu show tfplan

# Graph visualization
tofu graph | dot -Tsvg > graph.svg

# Force unlock state
tofu force-unlock LOCK_ID

# Refresh state
tofu refresh

# Show providers
tofu providers

# Console for testing expressions
tofu console

# Test infrastructure
tofu test

# Encrypt state (OpenTofu specific)
tofu state encrypt
```

## Common Patterns

### Multi-Environment Setup

```hcl
# environments/variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
}

variable "instance_config" {
  description = "Instance configuration by environment"
  type = map(object({
    instance_type = string
    min_size     = number
    max_size     = number
  }))
  default = {
    dev = {
      instance_type = "t3.micro"
      min_size     = 1
      max_size     = 2
    }
    staging = {
      instance_type = "t3.small"
      min_size     = 1
      max_size     = 3
    }
    prod = {
      instance_type = "t3.medium"
      min_size     = 2
      max_size     = 10
    }
  }
}

locals {
  env_config = var.instance_config[var.environment]
  
  common_tags = {
    Environment = var.environment
    ManagedBy   = "OpenTofu"
    Terraform   = false  # Indicate this is managed by OpenTofu
  }
}

resource "aws_launch_template" "app" {
  name_prefix   = "${var.environment}-app-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = local.env_config.instance_type
  
  tag_specifications {
    resource_type = "instance"
    tags = merge(local.common_tags, {
      Name = "${var.environment}-app-instance"
    })
  }
}
```

## Useful Commands Reference

| Command | Description |
|---------|-------------|
| `tofu init` | Initialize working directory |
| `tofu plan` | Create execution plan |
| `tofu apply` | Apply changes |
| `tofu destroy` | Destroy infrastructure |
| `tofu fmt` | Format configuration files |
| `tofu validate` | Validate configuration |
| `tofu show` | Show current state |
| `tofu output` | Show output values |
| `tofu refresh` | Update state with real infrastructure |
| `tofu import` | Import existing resource into state |
| `tofu state list` | List resources in state |
| `tofu state show` | Show resource in state |
| `tofu workspace list` | List workspaces |
| `tofu providers` | Show provider requirements |
| `tofu version` | Show OpenTofu version |
| `tofu test` | Run tests (OpenTofu specific) |
| `tofu state encrypt` | Encrypt state (OpenTofu specific) |

## Environment Variables

```bash
# OpenTofu configuration
export TF_CLI_CONFIG_FILE="$HOME/.tofurc"
export TF_DATA_DIR=".terraform"
export TF_INPUT=false
export TF_IN_AUTOMATION=true

# OpenTofu-specific variables
export TOFU_LOG=INFO
export TOFU_LOG_PATH="./tofu.log"
export TOFU_CLI_CONFIG_FILE="$HOME/.tofurc"

# Variables
export TF_VAR_region="us-west-2"
export TF_VAR_instance_type="t3.micro"
export TF_VAR_environment="production"

# Provider configuration
export AWS_REGION="us-west-2"
export AWS_PROFILE="default"
export GOOGLE_CREDENTIALS="path/to/credentials.json"
export ARM_SUBSCRIPTION_ID="subscription-id"

# State encryption
export TF_VAR_state_passphrase="your-secure-passphrase"
```

## Migration Guide

### From Terraform to OpenTofu

```bash
# 1. Install OpenTofu
brew install opentofu

# 2. Replace terraform commands with tofu
# No code changes needed - OpenTofu is fully compatible

# 3. Initialize with OpenTofu
tofu init

# 4. Optional: Enable state encryption
# Add encryption block to terraform {} block

# 5. Optional: Add testing
# Create .tftest.hcl files for your infrastructure

# 6. Optional: Update CI/CD pipelines
# Replace terraform commands with tofu commands
```

### Compatibility Notes

```hcl
# OpenTofu is compatible with:
# - All Terraform providers
# - All Terraform modules
# - Existing .tf files
# - Terraform state files

# OpenTofu adds:
# - State encryption
# - Enhanced testing framework
# - Provider-defined functions
# - Community governance
# - Improved error messages
```

---

*For more detailed information, visit the [official OpenTofu documentation](https://opentofu.org/docs/)*
