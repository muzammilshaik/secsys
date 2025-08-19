---
author: muju
title: "Terraform Proxmox LXC Containers"
description: "Learn how to provision and manage LXC containers using Terraform. This guide covers installation, configuration, and best practices."
categories: ["Linux"]
tags: ["Automation", "Terraform", "LXC", "DevOps"]
image:
  path: /assets/dev/tf/tflxc/tf-logo1.webp
  lqip: data:image/webp;base64,UklGRjwBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSJYAAAABgCNJkmqnzY8uMzN/sWShrXczMTMzM1zr4752dl+sbhARE/A3BY3k16gO8rWlg+Zk1dKkhAviuqgY+aSCFGc5AM0XkottBTk4Ct6bJInQWldBzlMKILrd0wAkqgNLHQncxulA0d0Iy8LizLgSnGd/JwZlmBwcGXfThylYDJNkf9cGyxLS839+oQPFK6EBqNyZGoDGswJWUDgggAAAAFAEAJ0BKhQAFAA+kUCYSaWjoiEoCqiwEglsAHjSIG0dt4rTAN5uAzTitYAA2m//Yww3TGemeBdksKYdQkbXNM+CP+XeJRKiT9+lNUmPT/+UF2Ih7KrEQ9lVbc3cpxQwXFTa5Lbxsbdebl/PyU95JuX8/JO7V8a4EOn/Nd6AAAAA
published: true
hidden: false
toc: true
# ref: https://www.tecmint.com/install-terraform-in-linux/
# ref: https://ronamosa.io/docs/engineer/LAB/proxmox-terraform/
---
<!-- 
Infrastructure as Code (IaC) has revolutionized the way we manage and deploy infrastructure. Terraform, one of the most popular IaC tools, enables automation and efficient provisioning of resources. In this guide, we'll walk through provisioning LXC containers using Terraform on a Debian server.

🏗️ Prerequisites


## Creating the user/role for terraform

Log into the Proxmox cluster or host using ssh (or mimic these in the GUI) then:
- Create a new role TerraformProv for the future terraform user.
- Create the user "terraform@pve"
- Add the TerraformProv role to the terraform user
- Create the api tocken for the terraform user

```bash
pveum role add TerraformProv -privs "Datastore.AllocateSpace Datastore.AllocateTemplate Datastore.Audit Pool.Allocate Sys.Audit Sys.Console Sys.Modify VM.Allocate VM.Audit VM.Clone VM.Config.CDROM VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Migrate VM.Monitor VM.PowerMgmt SDN.Use "
pveum user add terraform@pve --password <password>
pveum aclmod / -user terraform@pve -role TerraformProv
```

to modify the permissions

```bash
pveum role modify TerraformProv -privs "Datastore.AllocateSpace Datastore.AllocateTemplate Datastore.Audit Pool.Allocate Sys.Audit Sys.Console Sys.Modify VM.Allocate VM.Audit VM.Clone VM.Config.CDROM VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Migrate VM.Monitor VM.PowerMgmt SDN.Use User.Modify"
```

### API Token

From the proxmox GUI, create an API Token for the `terraform` user.

> note: ensure `privilege separation` for your API Token is disabled or terraform will error out later on:
{: .prompt-info }

![Terraform](/assets/dev/tf/tflxc/tf-api.png){: width="800" height="500" }

## 📌 Installing Terraform on Linux

Terraform is primarily distributed as a .zip package containing a single executable, which can be extracted and placed anywhere on your Linux system.

For seamless integration with configuration management tools, Terraform also provides official package repositories for Debian-based and RHEL-based distributions. This enables installation using standard package managers like APT, Yum, or DNF, simplifying updates and dependency management.

### Install Terraform in Debian, Ubuntu & Mint

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update 
sudo apt install terraform
```

### Install Terraform in RHEL and CentOS

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum update
sudo yum -y install terraform
```

### Install Terraform in Fedora

```bash
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo dnf update
sudo dnf -y install terraform
```

Now the installation can be verified by running a simple terraform version command.

```bash
terraform version
```

## 🛠️ Terraform Configuration

Terraform requires a provider and resource definitions. Below is a basic example of how to define LXC containers using Terraform.

### 📄 Terraform Configuration File (main.tf)

```bash
resource "proxmox_lxc" "basic" {
  target_node  = "pve"
  hostname     = "lxc-basic"
  ostemplate   = "local:vztmpl/ubuntu-20.04-standard_20.04-1_amd64.tar.gz"
  password     = "BasicLXCContainer"
  unprivileged = true

  // Terraform will crash without rootfs defined
  rootfs {
    storage = "local-zfs"
    size    = "8G"
  }

  network {
    name   = "eth0"
    bridge = "vmbr0"
    ip     = "dhcp"
  }
}
```

### 📌 Initializing Terraform

Run the following commands to initialize and apply the Terraform configuration:

```bash
terraform init
terraform apply -auto-approve
```

This will create the LXC container as defined in the `main.tf` file.


## 🎯 Managing LXC Containers with Terraform

Terraform allows easy management of LXC containers. Here are a few useful commands:

- Check Infrastructure State:
```bash
terraform show
```
- Update Configuration: Modify `main.tf` and apply changes:
```bash
terraform apply
```
- Destroy Containers:
```bash
terraform destroy -auto-approve
```

### Deploying the proxmox container securly 

Create a `provider.tf` file and install the below provider by running terraform init

```bash
terraform {
  required_providers {
    proxmox = {
      source  = "Telmate/proxmox"
      version = "3.0.1-rc6"
    }
  }
}


provider "proxmox" {
  pm_user              = var.proxmox_PM_USER
  pm_api_url           = var.proxmox_pm_api_url
  pm_api_token_id      = var.proxmox_PM_API_TOKEN_ID
  pm_api_token_secret  = var.proxmox_PM_API_TOKEN_SECRET
  pm_tls_insecure      = true
  pm_debug             = true
  pm_log_enable        = true
  pm_log_file          = "terraform-plugin-proxmox.log"
  pm_log_levels = {
    _default    = "debug"
    _capturelog = ""
  }
}

```

Create a `vars.tf` file and include your variables in it.

```bash
variable "proxmox_PM_USER" {
  type = string
}

variable "proxmox_pm_api_url" {
  type = string
}

variable "proxmox_PM_API_TOKEN_ID" {
  type = string
  sensitive = true
}

variable "proxmox_PM_API_TOKEN_SECRET" {
  type = string
  sensitive = true
}

```

Create a `terraform.tfvars` file and store your credentials in it. Remember to use `gitignore` if you are going to commit this to your public or private `GitHub` repo.

```bash
proxmox_PM_USER = "user@pve"
proxmox_pm_api_url = "https://$IP$:8006/api2/json"
proxmox_PM_API_TOKEN_ID = "user@pve!terraform"
proxmox_PM_API_TOKEN_SECRET = "abcedfg-b911-abcd-1234-791ebb4337f3"
```

Create a `main.tf` file, that is where I will a write a configuration rule that would provision two nodes in my proxmox home lab

```bash
resource "proxmox_lxc" "debain_lxc" {
  target_node  = "hulk"
  hostname     = "debian"
  ostemplate   = "nasp:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst"
  unprivileged = true

  ssh_public_keys = <<-EOT
    ssh-rsa <public_key_1> user@example.com
    ssh-ed25519 <public_key_2> user@example.com
  EOT

  // Terraform will crash without rootfs defined
  rootfs {
    storage = "local-lvm"
    size    = "10G"
  }

  network {
    name   = "eth0"
    bridge = "vmbr0"
    ip     = "XX.XX.XX.XXX"
    ip6    = "auto"
  }
}

```

Then run 

```bash
terraform plan
terraform apply
```

To clean the deployments 

```bash
terraform destroy
```



## ✅ Troubleshooting Steps

1️⃣ Ensure You Are Using the Correct -var-file Argument
Terraform does not automatically load custom *.tfvars files (except terraform.tfvars and *.auto.tfvars). You must specify it manually:
```bash
terraform plan -var-file="credentials.tfvars"
```

2️⃣ Validate Terraform’s Variable Recognition

Run the following command to confirm Terraform recognizes the variable:

```bash
terraform console
> var.proxmox_prod_secret_token
```

- If Terraform does not recognize it, the issue is with variable declaration or tfvars loading.
- If Terraform recognizes it, but terraform plan fails, check if the variable is being used correctly.

3️⃣ Recommended Ways to Load credentials.tfvars Automatically

- Use terraform.tfvars or *.auto.tfvars (Best Practice)
1️⃣ Terraform automatically loads:
    - terraform.tfvars
    - *.auto.tfvars (e.g., credentials.auto.tfvars)

2️⃣ Use an Environment Variable
Set the variables in your shell instead of credentials.tfvars:
```bash
export TF_VAR_proxmox_prod_secret_token="your-secret-token"
terraform plan
```
This method avoids `storing secrets` in files.

![Terraform](/assets/dev/tf/tflxc/tf-var-verify.png){: width="800" height="500" }

🔥 Conclusion

By integrating Terraform with LXC, you can automate container provisioning efficiently. This approach enhances reproducibility, scalability, and version control for your infrastructure. Explore more Terraform configurations and enhance your containerized environment!

For a quick reference on Terraform commands, check out the [Terraform Cheatsheet]({% post_url 2025-03-29-tfcheat %}){:target="_blank"} -->

# Complete Terraform Guide: From Basics to Advanced Multi-Provider Management

Infrastructure as Code (IaC) has revolutionized the way we manage and deploy infrastructure. Terraform, one of the most popular IaC tools, enables automation and efficient provisioning of resources across multiple cloud providers. This comprehensive guide covers everything from basic concepts to advanced configurations for AWS, Proxmox, GCP, and more.

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Terraform Installation](#terraform-installation)
3. [Core Terraform Concepts](#core-terraform-concepts)
4. [Variable Types and Usage](#variable-types-and-usage)
5. [Provider Configurations](#provider-configurations)
6. [Resource Management](#resource-management)
7. [Dynamic Blocks](#dynamic-blocks)
8. [State Management](#state-management)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)
11. [Multi-Provider Examples](#multi-provider-examples)

## 🛠️ Prerequisites

Before starting with Terraform, ensure you have:

- Basic understanding of cloud services (AWS, GCP, Azure)
- Command line familiarity
- Text editor or IDE
- Administrative access to your target infrastructure
- API credentials for your chosen providers

## 📦 Terraform Installation

### Install Terraform on Linux (Debian/Ubuntu)

```bash
# Add HashiCorp's GPG key
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# Add the repository
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# Update and install
sudo apt update && sudo apt install terraform
```

### Install Terraform on RHEL/CentOS

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum update && sudo yum -y install terraform
```

### Install Terraform on macOS

```bash
# Using Homebrew
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Or download binary directly
curl -LO https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_darwin_amd64.zip
```

### Verify Installation

```bash
terraform version
```

## 🎯 Core Terraform Concepts

### Terraform Workflow

```bash
# Initialize working directory
terraform init

# Validate configuration
terraform validate

# Plan changes
terraform plan

# Apply changes
terraform apply

# Destroy infrastructure
terraform destroy
```

### Essential Terraform Files

- **main.tf** - Primary configuration file
- **variables.tf** - Variable definitions
- **outputs.tf** - Output values
- **terraform.tfvars** - Variable values
- **provider.tf** - Provider configurations
- **versions.tf** - Terraform and provider version constraints

## 📊 Variable Types and Usage

Variables are fundamental to creating reusable and flexible Terraform configurations. Let's explore different variable types with practical examples.

### Demo 1: String Variables

**variables.tf**
```hcl
variable "environment" {
  description = "Environment name (dev, staging, prod)"
  type        = string
  default     = "dev"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "project_name" {
  description = "Name of the project"
  type        = string
  default     = "myapp"
}

variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}
```

**main.tf (String Variable Usage)**
```hcl
resource "aws_s3_bucket" "example" {
  bucket = "${var.project_name}-${var.environment}-bucket"
  
  tags = {
    Name        = "${var.project_name}-${var.environment}"
    Environment = var.environment
    Region      = var.region
  }
}
```

### Demo 2: List Variables

**variables.tf**
```hcl
variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

variable "allowed_ports" {
  description = "List of allowed ports for security group"
  type        = list(number)
  default     = [22, 80, 443, 3000, 8080]
}

variable "instance_types" {
  description = "List of EC2 instance types"
  type        = list(string)
  default     = ["t3.micro", "t3.small", "t3.medium"]
}
```

**main.tf (List Variable Usage)**
```hcl
resource "aws_subnet" "public" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 1}.0/24"
  availability_zone = var.availability_zones[count.index]
  
  tags = {
    Name = "Public Subnet ${count.index + 1}"
  }
}

resource "aws_security_group" "web" {
  name_prefix = "${var.project_name}-web-"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

### Demo 3: Map Variables

**variables.tf**
```hcl
variable "common_tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default = {
    Owner       = "DevOps Team"
    Project     = "Infrastructure"
    CostCenter  = "Engineering"
    Terraform   = "true"
  }
}

variable "instance_configs" {
  description = "Instance configuration per environment"
  type        = map(string)
  default = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "t3.large"
  }
}

variable "region_configs" {
  description = "Configuration per region"
  type = map(object({
    ami_id        = string
    instance_type = string
    key_name      = string
  }))
  default = {
    us-west-2 = {
      ami_id        = "ami-0c02fb55956c7d316"
      instance_type = "t3.micro"
      key_name      = "my-key-west"
    }
    us-east-1 = {
      ami_id        = "ami-0abcdef1234567890"
      instance_type = "t3.small"
      key_name      = "my-key-east"
    }
  }
}
```

**main.tf (Map Variable Usage)**
```hcl
resource "aws_instance" "web" {
  ami           = var.region_configs[var.region].ami_id
  instance_type = var.instance_configs[var.environment]
  key_name      = var.region_configs[var.region].key_name
  
  tags = merge(var.common_tags, {
    Name        = "${var.project_name}-${var.environment}-web"
    Environment = var.environment
  })
}

locals {
  environment_tags = {
    dev     = { Color = "green", Purpose = "development" }
    staging = { Color = "yellow", Purpose = "testing" }
    prod    = { Color = "red", Purpose = "production" }
  }
}

resource "aws_s3_bucket" "app_bucket" {
  bucket = "${var.project_name}-${var.environment}-data"
  
  tags = merge(
    var.common_tags,
    local.environment_tags[var.environment],
    {
      Name = "${var.project_name}-${var.environment}-bucket"
    }
  )
}
```

### Demo 4: Object Variables

**variables.tf**
```hcl
variable "database_config" {
  description = "Database configuration"
  type = object({
    engine            = string
    engine_version    = string
    instance_class    = string
    allocated_storage = number
    storage_encrypted = bool
    multi_az          = bool
    backup_retention  = number
    backup_window     = string
    maintenance_window = string
  })
  default = {
    engine            = "mysql"
    engine_version    = "8.0"
    instance_class    = "db.t3.micro"
    allocated_storage = 20
    storage_encrypted = true
    multi_az          = false
    backup_retention  = 7
    backup_window     = "03:00-04:00"
    maintenance_window = "sun:04:00-sun:05:00"
  }
}

variable "vpc_config" {
  description = "VPC configuration"
  type = object({
    cidr_block           = string
    enable_dns_hostnames = bool
    enable_dns_support   = bool
    public_subnets       = list(string)
    private_subnets      = list(string)
    availability_zones   = list(string)
  })
  default = {
    cidr_block           = "10.0.0.0/16"
    enable_dns_hostnames = true
    enable_dns_support   = true
    public_subnets       = ["10.0.1.0/24", "10.0.2.0/24"]
    private_subnets      = ["10.0.10.0/24", "10.0.20.0/24"]
    availability_zones   = ["us-west-2a", "us-west-2b"]
  }
}

variable "application_config" {
  description = "Application configuration with nested objects"
  type = object({
    name = string
    port = number
    health_check = object({
      path                = string
      interval            = number
      timeout             = number
      healthy_threshold   = number
      unhealthy_threshold = number
    })
    scaling = object({
      min_capacity = number
      max_capacity = number
      target_cpu   = number
    })
    environment_variables = map(string)
  })
  default = {
    name = "web-app"
    port = 3000
    health_check = {
      path                = "/health"
      interval            = 30
      timeout             = 5
      healthy_threshold   = 2
      unhealthy_threshold = 3
    }
    scaling = {
      min_capacity = 2
      max_capacity = 10
      target_cpu   = 70
    }
    environment_variables = {
      NODE_ENV = "production"
      LOG_LEVEL = "info"
    }
  }
}
```

**main.tf (Object Variable Usage)**
```hcl
# VPC using object variable
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_config.cidr_block
  enable_dns_hostnames = var.vpc_config.enable_dns_hostnames
  enable_dns_support   = var.vpc_config.enable_dns_support
  
  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-vpc"
  })
}

# Database using object variable
resource "aws_db_instance" "main" {
  identifier             = "${var.project_name}-${var.environment}-db"
  engine                 = var.database_config.engine
  engine_version         = var.database_config.engine_version
  instance_class         = var.database_config.instance_class
  allocated_storage      = var.database_config.allocated_storage
  storage_encrypted      = var.database_config.storage_encrypted
  multi_az               = var.database_config.multi_az
  backup_retention_period = var.database_config.backup_retention
  backup_window          = var.database_config.backup_window
  maintenance_window     = var.database_config.maintenance_window
  
  db_name  = "${var.project_name}_${var.environment}"
  username = "admin"
  password = "your-secure-password" # Use AWS Secrets Manager in production
  
  vpc_security_group_ids = [aws_security_group.database.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  skip_final_snapshot = true
  
  tags = var.common_tags
}

# Application Load Balancer with health check configuration
resource "aws_lb_target_group" "app" {
  name     = "${var.project_name}-${var.environment}-tg"
  port     = var.application_config.port
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  
  health_check {
    enabled             = true
    healthy_threshold   = var.application_config.health_check.healthy_threshold
    unhealthy_threshold = var.application_config.health_check.unhealthy_threshold
    timeout             = var.application_config.health_check.timeout
    interval            = var.application_config.health_check.interval
    path                = var.application_config.health_check.path
    matcher             = "200"
    port                = "traffic-port"
    protocol            = "HTTP"
  }
  
  tags = var.common_tags
}
```

## 🔄 Dynamic Blocks

Dynamic blocks allow you to dynamically construct repeatable nested blocks based on complex variable types.

### Basic Dynamic Block Example

**variables.tf**
```hcl
variable "security_group_rules" {
  description = "Security group rules"
  type = list(object({
    type        = string
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
    description = string
  }))
  default = [
    {
      type        = "ingress"
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["10.0.0.0/16"]
      description = "SSH from VPC"
    },
    {
      type        = "ingress"
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTP from anywhere"
    },
    {
      type        = "ingress"
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS from anywhere"
    }
  ]
}
```

**main.tf**
```hcl
resource "aws_security_group" "web" {
  name_prefix = "${var.project_name}-${var.environment}-web-"
  vpc_id      = aws_vpc.main.id
  
  dynamic "ingress" {
    for_each = [for rule in var.security_group_rules : rule if rule.type == "ingress"]
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }
  
  dynamic "egress" {
    for_each = [for rule in var.security_group_rules : rule if rule.type == "egress"]
    content {
      from_port   = egress.value.from_port
      to_port     = egress.value.to_port
      protocol    = egress.value.protocol
      cidr_blocks = egress.value.cidr_blocks
      description = egress.value.description
    }
  }
  
  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-web-sg"
  })
}
```

### Advanced Dynamic Block with Nested Objects

**variables.tf**
```hcl
variable "load_balancer_listeners" {
  description = "Load balancer listener configurations"
  type = list(object({
    port     = number
    protocol = string
    ssl_policy = optional(string)
    certificate_arn = optional(string)
    default_actions = list(object({
      type             = string
      target_group_arn = optional(string)
      redirect = optional(object({
        port        = string
        protocol    = string
        status_code = string
      }))
    }))
  }))
  default = [
    {
      port     = 80
      protocol = "HTTP"
      default_actions = [
        {
          type = "redirect"
          redirect = {
            port        = "443"
            protocol    = "HTTPS"
            status_code = "HTTP_301"
          }
        }
      ]
    },
    {
      port            = 443
      protocol        = "HTTPS"
      ssl_policy      = "ELBSecurityPolicy-TLS-1-2-2017-01"
      certificate_arn = "arn:aws:acm:region:account:certificate/certificate-id"
      default_actions = [
        {
          type             = "forward"
          target_group_arn = "arn:aws:elasticloadbalancing:region:account:targetgroup/name"
        }
      ]
    }
  ]
}
```

**main.tf**
```hcl
resource "aws_lb_listener" "web" {
  count             = length(var.load_balancer_listeners)
  load_balancer_arn = aws_lb.main.arn
  port              = var.load_balancer_listeners[count.index].port
  protocol          = var.load_balancer_listeners[count.index].protocol
  ssl_policy        = var.load_balancer_listeners[count.index].ssl_policy
  certificate_arn   = var.load_balancer_listeners[count.index].certificate_arn
  
  dynamic "default_action" {
    for_each = var.load_balancer_listeners[count.index].default_actions
    content {
      type             = default_action.value.type
      target_group_arn = default_action.value.target_group_arn
      
      dynamic "redirect" {
        for_each = default_action.value.redirect != null ? [default_action.value.redirect] : []
        content {
          port        = redirect.value.port
          protocol    = redirect.value.protocol
          status_code = redirect.value.status_code
        }
      }
    }
  }
}
```

## 🔧 Provider Configurations

### AWS Provider

**provider.tf**
```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      ManagedBy   = "Terraform"
      Environment = var.environment
      Project     = var.project_name
    }
  }
}
```

**variables.tf (AWS)**
```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}

variable "aws_profile" {
  description = "AWS CLI profile"
  type        = string
  default     = "default"
}
```

### Proxmox Provider

**provider.tf (Proxmox)**
```hcl
terraform {
  required_providers {
    proxmox = {
      source  = "Telmate/proxmox"
      version = "~> 2.9"
    }
  }
}

provider "proxmox" {
  pm_user              = var.proxmox_user
  pm_api_url           = var.proxmox_api_url
  pm_api_token_id      = var.proxmox_token_id
  pm_api_token_secret  = var.proxmox_token_secret
  pm_tls_insecure      = var.proxmox_tls_insecure
  pm_debug             = var.proxmox_debug
  pm_log_enable        = true
  pm_log_file          = "terraform-plugin-proxmox.log"
  pm_log_levels = {
    _default    = "debug"
    _capturelog = ""
  }
}
```

**variables.tf (Proxmox)**
```hcl
variable "proxmox_user" {
  description = "Proxmox user"
  type        = string
  sensitive   = true
}

variable "proxmox_api_url" {
  description = "Proxmox API URL"
  type        = string
}

variable "proxmox_token_id" {
  description = "Proxmox API token ID"
  type        = string
  sensitive   = true
}

variable "proxmox_token_secret" {
  description = "Proxmox API token secret"
  type        = string
  sensitive   = true
}

variable "proxmox_tls_insecure" {
  description = "Skip TLS verification"
  type        = bool
  default     = true
}

variable "proxmox_debug" {
  description = "Enable debug logging"
  type        = bool
  default     = false
}
```

### Google Cloud Provider

**provider.tf (GCP)**
```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
}

provider "google" {
  project = var.gcp_project_id
  region  = var.gcp_region
  zone    = var.gcp_zone
}
```

**variables.tf (GCP)**
```hcl
variable "gcp_project_id" {
  description = "GCP project ID"
  type        = string
}

variable "gcp_region" {
  description = "GCP region"
  type        = string
  default     = "us-central1"
}

variable "gcp_zone" {
  description = "GCP zone"
  type        = string
  default     = "us-central1-a"
}
```

## 📝 Complete Multi-Provider Examples

### AWS Complete Example

**main.tf (AWS)**
```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_config.cidr_block
  enable_dns_hostnames = var.vpc_config.enable_dns_hostnames
  enable_dns_support   = var.vpc_config.enable_dns_support
  
  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-vpc"
  })
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-igw"
  })
}

# Public Subnets
resource "aws_subnet" "public" {
  count             = length(var.vpc_config.public_subnets)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.vpc_config.public_subnets[count.index]
  availability_zone = var.vpc_config.availability_zones[count.index]
  
  map_public_ip_on_launch = true
  
  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-public-${count.index + 1}"
    Type = "Public"
  })
}

# Route Table for Public Subnets
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-public-rt"
  })
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# Security Group
resource "aws_security_group" "web" {
  name_prefix = "${var.project_name}-${var.environment}-web-"
  vpc_id      = aws_vpc.main.id
  
  dynamic "ingress" {
    for_each = [for rule in var.security_group_rules : rule if rule.type == "ingress"]
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "All outbound traffic"
  }
  
  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-web-sg"
  })
}

# EC2 Instance
resource "aws_instance" "web" {
  count                  = var.instance_count
  ami                    = var.region_configs[var.aws_region].ami_id
  instance_type          = var.instance_configs[var.environment]
  key_name              = var.region_configs[var.aws_region].key_name
  vpc_security_group_ids = [aws_security_group.web.id]
  subnet_id             = aws_subnet.public[count.index % length(aws_subnet.public)].id
  
  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Hello from ${var.project_name}-${var.environment}-${count.index + 1}</h1>" > /var/www/html/index.html
  EOF
  
  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-web-${count.index + 1}"
  })
}
```

### Proxmox Complete Example

**main.tf (Proxmox)**
```hcl
# LXC Container
resource "proxmox_lxc" "container" {
  count       = var.container_count
  target_node = var.proxmox_node
  hostname    = "${var.project_name}-${var.environment}-${count.index + 1}"
  ostemplate  = var.container_template
  password    = var.container_password
  unprivileged = true
  
  ssh_public_keys = var.ssh_public_keys
  
  # Root filesystem
  rootfs {
    storage = var.storage_pool
    size    = var.container_disk_size
  }
  
  # Network configuration
  network {
    name   = "eth0"
    bridge = var.network_bridge
    ip     = var.use_dhcp ? "dhcp" : "${var.network_base}.${count.index + 10}/24"
    gw     = var.use_dhcp ? null : var.gateway_ip
  }
  
  # Memory and CPU
  memory = var.container_memory
  cores  = var.container_cores
  
  # Features
  features {
    nesting = var.enable_nesting
    mount   = var.enable_mount
  }
  
  # Startup configuration
  startup {
    order      = count.index + 1
    up_delay   = 30
    down_delay = 30
  }
  
  lifecycle {
    ignore_changes = [
      network[0].hwaddr,
    ]
  }
}

# VM (if needed)
resource "proxmox_vm_qemu" "vm" {
  count       = var.vm_count
  target_node = var.proxmox_node
  name        = "${var.project_name}-${var.environment}-vm-${count.index + 1}"
  desc        = "VM created by Terraform"
  
  # VM configuration
  clone      = var.vm_template
  full_clone = true
  
  # Hardware
  cores   = var.vm_cores
  sockets = 1
  memory  = var.vm_memory
  
  # Disk
  disk {
    slot    = 0
    size    = var.vm_disk_size
    type    = "scsi"
    storage = var.storage_pool
    ssd     = 1
  }
  
  # Network
  network {
    model  = "virtio"
    bridge = var.network_bridge
  }
  
  # Cloud-init
  os_type    = "cloud-init"
  ciuser     = var.vm_user
  cipassword = var.vm_password
  sshkeys    = var.ssh_public_keys
  
  # IP configuration
  ipconfig0 = var.use_dhcp ? "ip=dhcp" : "ip=${var.network_base}.${count.index + 100}/24,gw=${var.gateway_ip}"
  
  lifecycle {
    ignore_changes = [
      network,
    ]
  }
}
```

### GCP Complete Example

**main.tf (GCP)**
```hcl
# VPC Network
resource "google_compute_network" "vpc" {
  name                    = "${var.project_name}-${var.environment}-vpc"
  auto_create_subnetworks = false
  description             = "VPC for ${var.project_name} ${var.environment}"
}

# Subnet
resource "google_compute_subnetwork" "subnet" {
  count         = length(var.gcp_subnets)
  name          = "${var.project_name}-${var.environment}-subnet-${count.index + 1}"
  ip_cidr_range = var.gcp_subnets[count.index]
  region        = var.gcp_region
  network       = google_compute_network.vpc.id
  
  secondary_ip_range {
    range_name    = "pods"
    ip_cidr_range = var.gcp_pod_ranges[count.index]
  }
  
  secondary_ip_range {
    range_name    = "services"
    ip_cidr_range = var.gcp_service_ranges[count.index]
  }
}

# Firewall Rules
resource "google_compute_firewall" "web" {
  name    = "${var.project_name}-${var.environment}-web"
  network = google_compute_network.vpc.name
  
  allow {
    protocol = "tcp"
    ports    = ["22", "80", "443"]
  }
  
  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web-server"]
}

# Compute Instance
resource "google_compute_instance" "web" {
  count        = var.gcp_instance_count
  name         = "${var.project_name}-${var.environment}-web-${count.index + 1}"
  machine_type = var.gcp_machine_type
  zone         = var.gcp_zone
  
  boot_disk {
    initialize_params {
      image = var.gcp_image
      size  = var.gcp_disk_size
      type  = "pd-standard"
    }
  }
  
  network_interface {
    network    = google_compute_network.vpc.id
    subnetwork = google_compute_subnetwork.subnet[0].id
    
    access_config {
      // Ephemeral public IP
    }
  }
  
  metadata_startup_script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
    systemctl enable nginx
    echo "<h1>Hello from ${var.project_name}-${var.environment}-${count.index + 1}</h1>" > /var/www/html/index.html
  EOF
  
  tags = ["web-server"]
  
  labels = {
    environment = var.environment
    project     = var.project_name
    managed_by  = "terraform"
  }
}

# Load Balancer
resource "google_compute_global_address" "lb_ip" {
  name = "${var.project_name}-${var.environment}-lb-ip"
}

resource "google_compute_health_check" "web" {
  name = "${var.project_name}-${var.environment}-health-check"
  
  http_health_check {
    port_specification = "USE_FIXED_PORT"
    port               = 80
    request_path       = "/"
  }
}

resource "google_compute_backend_service" "web" {
  name        = "${var.project_name}-${var.environment}-backend"
  port_name   = "http"
  protocol    = "HTTP"
  timeout_sec = 30
  
  health_checks = [google_compute_health_check.web.id]
  
  backend {
    group = google_compute_instance_group.web.id
  }
}

resource "google_compute_instance_group" "web" {
  name = "${var.project_name}-${var.environment}-instance-group"
  zone = var.gcp_zone
  
  instances = google_compute_instance.web[*].id
  
  named_port {
    name = "http"
    port = "80"
  }
}
```

## 🔧 Variable Files and Environment Management

### terraform.tfvars (Development)
```hcl
# Development Environment Configuration
environment    = "dev"
project_name   = "myapp"
aws_region     = "us-west-2"
instance_count = 1

# VPC Configuration
vpc_config = {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  public_subnets       = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets      = ["10.0.10.0/24", "10.0.20.0/24"]
  availability_zones   = ["us-west-2a", "us-west-2b"]
}

# Common Tags
common_tags = {
  Owner       = "DevOps Team"
  Project     = "MyApp"
  Environment = "dev"
  CostCenter  = "Engineering"
  Terraform   = "true"
}

# Security Group Rules
security_group_rules = [
  {
    type        = "ingress"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
    description = "SSH from VPC"
  },
  {
    type        = "ingress"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTP from anywhere"
  }
]

# Database Configuration
database_config = {
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  storage_encrypted = true
  multi_az          = false
  backup_retention  = 7
  backup_window     = "03:00-04:00"
  maintenance_window = "sun:04:00-sun:05:00"
}
```

### production.tfvars (Production)
```hcl
# Production Environment Configuration
environment    = "prod"
project_name   = "myapp"
aws_region     = "us-west-2"
instance_count = 3

# VPC Configuration
vpc_config = {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  public_subnets       = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  private_subnets      = ["10.0.10.0/24", "10.0.20.0/24", "10.0.30.0/24"]
  availability_zones   = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

# Instance Types per Environment
instance_configs = {
  dev     = "t3.micro"
  staging = "t3.small"
  prod    = "t3.large"
}

# Database Configuration for Production
database_config = {
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.r5.large"
  allocated_storage = 100
  storage_encrypted = true
  multi_az          = true
  backup_retention  = 30
  backup_window     = "03:00-04:00"
  maintenance_window = "sun:04:00-sun:05:00"
}
```

### proxmox.tfvars
```hcl
# Proxmox Configuration
proxmox_user         = "terraform@pve"
proxmox_api_url      = "https://192.168.1.100:8006/api2/json"
proxmox_token_id     = "terraform@pve!terraform"
proxmox_token_secret = "your-api-token-secret"
proxmox_tls_insecure = true

# Node and Storage Configuration
proxmox_node       = "pve"
storage_pool       = "local-zfs"
network_bridge     = "vmbr0"
container_template = "local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst"

# Container Configuration
container_count      = 2
container_memory     = 1024
container_cores      = 2
container_disk_size  = "10G"
container_password   = "SecurePassword123!"
enable_nesting       = true
enable_mount         = "nfs,cifs"

# Network Configuration
use_dhcp     = false
network_base = "192.168.1"
gateway_ip   = "192.168.1.1"

# SSH Keys
ssh_public_keys = <<-EOT
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... user@example.com
  ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGq... user@example.com
EOT
```

## 🎯 State Management

### Local State (Development)
```hcl
# versions.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### Remote State (Production)
```hcl
# versions.tf with remote state
terraform {
  required_version = ">= 1.0"
  
  backend "s3" {
    bucket         = "myapp-terraform-state"
    key            = "environments/prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### State Management Commands
```bash
# Initialize with remote state
terraform init

# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Move resource in state
terraform state mv aws_instance.web aws_instance.web_server

# Remove resource from state
terraform state rm aws_instance.web

# Show state
terraform state list
terraform state show aws_instance.web

# Refresh state
terraform refresh

# Workspace management
terraform workspace new staging
terraform workspace select prod
terraform workspace list
```

## 📋 outputs.tf Examples

```hcl
# outputs.tf
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}

output "instance_public_ips" {
  description = "Public IP addresses of EC2 instances"
  value       = aws_instance.web[*].public_ip
}

output "instance_private_ips" {
  description = "Private IP addresses of EC2 instances"
  value       = aws_instance.web[*].private_ip
}

output "load_balancer_dns" {
  description = "DNS name of the load balancer"
  value       = aws_lb.main.dns_name
}

output "database_endpoint" {
  description = "RDS instance endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true
}

# Complex output with formatting
output "instance_details" {
  description = "Detailed information about instances"
  value = {
    for i, instance in aws_instance.web : 
    "web-${i + 1}" => {
      id         = instance.id
      public_ip  = instance.public_ip
      private_ip = instance.private_ip
      az         = instance.availability_zone
      type       = instance.instance_type
    }
  }
}

# Conditional output
output "ssl_certificate_arn" {
  description = "ARN of the SSL certificate"
  value       = var.environment == "prod" ? aws_acm_certificate.main[0].arn : null
}

# Proxmox outputs
output "lxc_container_ips" {
  description = "IP addresses of LXC containers"
  value       = proxmox_lxc.container[*].network[0].ip
}

output "vm_ips" {
  description = "IP addresses of VMs"
  value       = proxmox_vm_qemu.vm[*].default_ipv4_address
}
```

## 🚀 Advanced Terraform Techniques

### Modules Structure
```bash
# Directory structure
modules/
├── aws-vpc/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── versions.tf
├── aws-ec2/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── versions.tf
└── proxmox-lxc/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── versions.tf
```

### Using Modules
```hcl
# main.tf using modules
module "vpc" {
  source = "./modules/aws-vpc"
  
  project_name = var.project_name
  environment  = var.environment
  vpc_config   = var.vpc_config
  common_tags  = var.common_tags
}

module "ec2_instances" {
  source = "./modules/aws-ec2"
  
  count = var.create_instances ? 1 : 0
  
  project_name         = var.project_name
  environment          = var.environment
  vpc_id              = module.vpc.vpc_id
  public_subnet_ids   = module.vpc.public_subnet_ids
  security_group_rules = var.security_group_rules
  instance_count      = var.instance_count
  common_tags         = var.common_tags
  
  depends_on = [module.vpc]
}

module "proxmox_containers" {
  source = "./modules/proxmox-lxc"
  
  count = var.create_proxmox_resources ? 1 : 0
  
  proxmox_node       = var.proxmox_node
  project_name       = var.project_name
  environment        = var.environment
  container_count    = var.container_count
  container_config   = var.container_config
}
```

### Data Sources
```hcl
# data.tf
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

data "aws_caller_identity" "current" {}

data "aws_region" "current" {}

# Using data sources
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  
  tags = {
    Name      = "${var.project_name}-web"
    AccountId = data.aws_caller_identity.current.account_id
    Region    = data.aws_region.current.name
  }
}
```

### Locals and Functions
```hcl
# locals.tf
locals {
  # Environment-specific configurations
  environment_config = {
    dev = {
      instance_type = "t3.micro"
      min_size      = 1
      max_size      = 2
      disk_size     = 20
    }
    staging = {
      instance_type = "t3.small"
      min_size      = 2
      max_size      = 4
      disk_size     = 40
    }
    prod = {
      instance_type = "t3.medium"
      min_size      = 3
      max_size      = 10
      disk_size     = 80
    }
  }
  
  # Current environment configuration
  current_config = local.environment_config[var.environment]
  
  # Common tags with computed values
  common_tags = merge(var.common_tags, {
    Environment   = var.environment
    CreatedBy     = "terraform"
    LastModified  = timestamp()
    AccountId     = data.aws_caller_identity.current.account_id
    Region        = data.aws_region.current.name
  })
  
  # Network calculations
  vpc_cidr        = var.vpc_config.cidr_block
  subnet_bits     = 8
  subnet_count    = length(var.vpc_config.availability_zones)
  public_subnets  = [for i in range(local.subnet_count) : cidrsubnet(local.vpc_cidr, local.subnet_bits, i)]
  private_subnets = [for i in range(local.subnet_count) : cidrsubnet(local.vpc_cidr, local.subnet_bits, i + 10)]
  
  # Security group rules processing
  ingress_rules = {
    for idx, rule in var.security_group_rules :
    "${rule.from_port}-${rule.to_port}-${rule.protocol}" => rule
    if rule.type == "ingress"
  }
  
  # Instance user data
  user_data = base64encode(templatefile("${path.module}/user_data.sh", {
    project_name = var.project_name
    environment  = var.environment
    app_version  = var.app_version
  }))
}
```

## 🔍 Best Practices

### File Organization
```bash
# Recommended project structure
terraform-infrastructure/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── outputs.tf
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── staging.tfvars
│   │   └── outputs.tf
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── prod.tfvars
│       └── outputs.tf
├── modules/
│   ├── vpc/
│   ├── ec2/
│   └── rds/
├── global/
│   ├── s3-backend/
│   └── iam/
└── scripts/
    ├── deploy.sh
    └── destroy.sh
```

### Naming Conventions
```hcl
# Resource naming
resource "aws_instance" "web" {
  # Use descriptive names
  name = "${var.project_name}-${var.environment}-web-${count.index + 1}"
}

# Variable naming
variable "vpc_cidr_block" {         # Clear and descriptive
  # ...
}

variable "enable_monitoring" {       # Use enable/disable prefix for booleans
  type = bool
  # ...
}

variable "instance_count" {          # Use count suffix for numbers
  type = number
  # ...
}
```

### Security Best Practices
```hcl
# variables.tf
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true  # Mark sensitive variables
}

variable "api_keys" {
  description = "API keys"
  type        = map(string)
  sensitive   = true
}

# Use AWS Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name        = "${var.project_name}-${var.environment}-db-password"
  description = "Database password for ${var.project_name}"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = var.db_password
}

# Reference secrets in RDS
resource "aws_db_instance" "main" {
  # ... other configuration
  manage_master_user_password = true
}
```

## 🔧 Troubleshooting Guide

### Common Issues and Solutions

#### 1. Provider Authentication Issues
```bash
# AWS credentials
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-west-2"

# Or use AWS CLI profile
aws configure --profile terraform
export AWS_PROFILE=terraform

# Verify credentials
aws sts get-caller-identity
```

#### 2. State Lock Issues
```bash
# Force unlock (use carefully)
terraform force-unlock LOCK_ID

# If DynamoDB table is locked
aws dynamodb scan --table-name terraform-locks
```

#### 3. Variable Validation Errors
```bash
# Check variable values
terraform console
> var.environment
> var.vpc_config

# Validate configuration
terraform validate

# Plan with specific var file
terraform plan -var-file="prod.tfvars"
```

#### 4. Resource Import
```bash
# Import existing AWS VPC
terraform import aws_vpc.main vpc-12345678

# Import Proxmox container
terraform import proxmox_lxc.container node/lxc/100
```

### Debugging Techniques
```bash
# Enable debug logging
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log

# Trace specific operations
export TF_LOG=TRACE terraform apply

# Check state file
terraform state list
terraform state show resource_name

# Format and validate
terraform fmt -recursive
terraform validate
```

## 📚 Essential Terraform Commands Reference

### Basic Commands
```bash
# Initialize and download providers
terraform init

# Upgrade providers
terraform init -upgrade

# Validate configuration
terraform validate

# Format code
terraform fmt -recursive

# Plan changes
terraform plan
terraform plan -var-file="prod.tfvars"
terraform plan -target=aws_instance.web

# Apply changes
terraform apply
terraform apply -auto-approve
terraform apply -var-file="prod.tfvars"
terraform apply -target=aws_instance.web

# Destroy resources
terraform destroy
terraform destroy -auto-approve
terraform destroy -target=aws_instance.web
```

### State Management
```bash
# List resources in state
terraform state list

# Show resource details
terraform state show aws_instance.web

# Move resources
terraform state mv aws_instance.web aws_instance.web_server

# Remove from state
terraform state rm aws_instance.web

# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Pull remote state
terraform state pull

# Push local state
terraform state push
```

### Workspace Management
```bash
# List workspaces
terraform workspace list

# Create workspace
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Select workspace
terraform workspace select dev

# Delete workspace
terraform workspace delete staging
```

## 🎯 Deployment Scripts

### deploy.sh
```bash
#!/bin/bash

set -e

ENVIRONMENT=$1
VAR_FILE="${ENVIRONMENT}.tfvars"

if [ -z "$ENVIRONMENT" ]; then
    echo "Usage: $0 <environment>"
    echo "Example: $0 dev"
    exit 1
fi

if [ ! -f "$VAR_FILE" ]; then
    echo "Variable file $VAR_FILE not found!"
    exit 1
fi

echo "Deploying to $ENVIRONMENT environment..."

# Initialize Terraform
terraform init

# Select workspace
terraform workspace select $ENVIRONMENT || terraform workspace new $ENVIRONMENT

# Validate configuration
terraform validate

# Plan deployment
echo "Planning deployment..."
terraform plan -var-file="$VAR_FILE" -out="$ENVIRONMENT.tfplan"

# Ask for confirmation
read -p "Do you want to apply these changes? (yes/no): " confirm
if [ "$confirm" = "yes" ]; then
    echo "Applying changes..."
    terraform apply "$ENVIRONMENT.tfplan"
    echo "Deployment completed successfully!"
else
    echo "Deployment cancelled."
fi

# Clean up plan file
rm -f "$ENVIRONMENT.tfplan"
```

### destroy.sh
```bash
#!/bin/bash

set -e

ENVIRONMENT=$1
VAR_FILE="${ENVIRONMENT}.tfvars"

if [ -z "$ENVIRONMENT" ]; then
    echo "Usage: $0 <environment>"
    echo "Example: $0 dev"
    exit 1
fi

echo "⚠️  WARNING: This will destroy all resources in $ENVIRONMENT environment!"
read -p "Are you sure you want to continue? (type 'yes' to confirm): " confirm

if [ "$confirm" != "yes" ]; then
    echo "Destruction cancelled."
    exit 1
fi

echo "Destroying $ENVIRONMENT environment..."

# Select workspace
terraform workspace select $ENVIRONMENT

# Destroy resources
terraform destroy -var-file="$VAR_FILE" -auto-approve

echo "Environment $ENVIRONMENT has been destroyed!"
```

## 🏁 Conclusion

This comprehensive Terraform guide provides you with everything needed to manage infrastructure across multiple providers. The examples cover:

- **Variable Types**: String, List, Map, and Object variables with real-world examples
- **Dynamic Blocks**: Advanced configuration patterns for complex resources
- **Multi-Provider Support**: Complete examples for AWS, Proxmox, and GCP
- **State Management**: Local and remote state configurations
- **Best Practices**: Security, naming conventions, and project organization
- **Troubleshooting**: Common issues and debugging techniques

### Quick Reference Summary

1. **Always use version constraints** for providers
2. **Implement remote state** for production environments
3. **Use variables and locals** for reusable configurations
4. **Apply consistent naming** conventions
5. **Validate and format** code regularly
6. **Use modules** for reusable components
7. **Mark sensitive variables** appropriately
8. **Test in dev/staging** before production deployment

### Next Steps

- Implement CI/CD pipelines with Terraform
- Explore Terraform Cloud for team collaboration
- Study advanced patterns like the count and for_each meta-arguments
- Learn about Terraform testing with `terraform test`
- Explore policy as code with Sentinel or OPA

Keep this guide as your reference for Terraform development and operations across different cloud providers and infrastructure platforms!