---
author: muju
title: "Terraform Kickstart"
date: 2025-08-19 12:00:00 +0530
description: "Complete Terraform Mastery Guide: From basic to advanced level"
categories: ["Devops"]
tags: ["Automation", "Terraform", "LXC", "DevOps"]
image:
  path: /assets/dev/tf/tflxc/tf-logo1.webp
  lqip: data:image/webp;base64,UklGRjwBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSJYAAAABgCNJkmqnzY8uMzN/sWShrXczMTMzM1zr4752dl+sbhARE/A3BY3k16gO8rWlg+Zk1dKkhAviuqgY+aSCFGc5AM0XkottBTk4Ct6bJInQWldBzlMKILrd0wAkqgNLHQncxulA0d0Iy8LizLgSnGd/JwZlmBwcGXfThylYDJNkf9cGyxLS839+oQPFK6EBqNyZGoDGswJWUDgggAAAAFAEAJ0BKhQAFAA+kUCYSaWjoiEoCqiwEglsAHjSIG0dt4rTAN5uAzTitYAA2m//Yww3TGemeBdksKYdQkbXNM+CP+XeJRKiT9+lNUmPT/+UF2Ih7KrEQ9lVbc3cpxQwXFTa5Lbxsbdebl/PyU95JuX8/JO7V8a4EOn/Nd6AAAAA
published: true
hidden: false
toc: true
---

## Introduction to Terraform

Terraform is an Infrastructure as Code (IaC) tool created by HashiCorp that allows you to define and provision infrastructure using a declarative configuration language. With Terraform, you can manage resources across multiple cloud providers and services through a consistent workflow.

### Key Benefits:
- **Infrastructure as Code**: Version control your infrastructure
- **Multi-Cloud Support**: Works with AWS, Azure, GCP, and many others
- **Declarative**: Describe what you want, not how to get there
- **State Management**: Keeps track of your infrastructure
- **Plan & Apply**: Preview changes before applying them

### Core Concepts

1. **Providers**: Plugins that interact with cloud platforms or services
2. **Resources**: Infrastructure objects managed by Terraform (e.g., VMs, networks)
3. **Data Sources**: Read-only information fetched from providers
4. **Variables**: Parameterize your configurations
5. **Outputs**: Return values from your infrastructure
6. **Modules**: Reusable components
7. **State**: Terraform's record of managed resources

### Installation

#### Windows

```powershell
# Using Chocolatey
choco install terraform

# Using Scoop
scoop install terraform

# Manual installation
# 1. Download from https://www.terraform.io/downloads.html
# 2. Extract the zip file
# 3. Add to PATH environment variable
```

#### Linux

```bash
# Using apt (Ubuntu/Debian)
sudo apt update
sudo apt install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform

# Using yum (RHEL/CentOS)
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum install terraform
```

#### macOS

```bash
# Using Homebrew
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

#### Manual Installation:
1. Download from [terraform.io](https://terraform.io/downloads){:target="_blank"}
2. Extract the binary
3. Add to your PATH

### Verify Installation

```bash
terraform version
```

## TF Basics

### File Structure:
```
project/
├── main.tf          # Main configuration
├── variables.tf     # Variable definitions
├── outputs.tf       # Output definitions
├── providers.tf     # Provider configurations
├── terraform.tfvars # Variable values
└── modules/         # Custom modules
    └── vpc/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Basic Workflow

```bash
terraform init      # Initialize working directory
terraform plan       # Preview changes
terraform apply      # Apply changes
terraform destroy    # Destroy infrastructure
terraform validate   # Validate configuration
terraform fmt        # Format configuration files
terraform show       # Show current state
terraform output     # Show output values
```

### TF Configuration

Create a file named `main.tf`:

```hcl
# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "example" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "example-vpc"
  }
}
```

## TF Variables

Variables make your configurations flexible and reusable. They can be defined in multiple ways and support various data types.

### Variable Types

Terraform supports several variable types:

- **String**: Text values
- **Number**: Numeric values
- **Bool**: Boolean values (true/false)
- **List**: Ordered collection of values
- **Map**: Collection of key-value pairs
- **Set**: Unordered collection of unique values
- **Object**: Complex structured data
- **Tuple**: Fixed-length collection of values with potentially different types

### Variable Declaration Syntax:
```hcl
variable "variable_name" {
  description = "Description of the variable"
  type        = string  # string, number, bool, list, map, object, tuple, set
  default     = "default_value"
  sensitive   = false   # Hide value in logs
  validation {
    condition     = length(var.variable_name) > 4
    error_message = "Variable must be more than 4 characters."
  }
}
```

### Demo 1: Variable with String

Create a file named `variables.tf`:

```hcl
variable "instance_name" {
  description = "Name of the EC2 instance"
  type        = string
  default     = "my-instance"
  
  validation {
    condition     = length(var.instance_name) > 0
    error_message = "Instance name cannot be empty."
  }
}

variable "aws_region" {
  description = "AWS region for resources"
  type        = string
  default     = "us-west-2"
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

Use it in `main.tf`:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t2.micro"
  
  tags = {
    Name        = var.instance_name
    Environment = var.environment
  }
}
```

terraform.tfvars

```hcl
instance_name = "web-server-01"
aws_region    = "us-east-1"
environment   = "dev"
```

### Demo 2: Variable with List

In `variables.tf`:

```hcl
variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

variable "instance_types" {
  description = "List of instance types"
  type        = list(string)
  default     = ["t2.micro", "t2.small", "t2.medium"]
}

variable "subnet_cidrs" {
  description = "CIDR blocks for subnets"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
}
```

Use it in `main.tf`:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "main-vpc"
  }
}

resource "aws_subnet" "public" {
  count             = length(var.subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${count.index + 1}"
    Type = "Public"
  }
}

resource "aws_instance" "web" {
  count           = length(var.instance_types)
  ami             = "ami-0c55b159cbfafe1d0"
  instance_type   = var.instance_types[count.index]
  subnet_id       = aws_subnet.public[count.index].id
  
  tags = {
    Name = "web-server-${count.index + 1}"
  }
}
```

### Demo 3: Variable with Map

In `variables.tf`:

```hcl
variable "instance_config" {
  description = "Map of instance configurations"
  type        = map(string)
  default = {
    ami           = "ami-0c55b159cbfafe1d0"
    instance_type = "t2.micro"
    key_name      = "my-key"
  }
}

variable "tags" {
  description = "Map of tags to apply to resources"
  type        = map(string)
  default = {
    Environment = "dev"
    Owner       = "devops-team"
    Project     = "web-app"
  }
}

variable "port_mappings" {
  description = "Map of port configurations"
  type        = map(number)
  default = {
    http  = 80
    https = 443
    ssh   = 22
  }
}
```

Use it in `main.tf`:

```hcl
resource "aws_instance" "web" {
  ami           = var.instance_config["ami"]
  instance_type = var.instance_config["instance_type"]
  key_name      = var.instance_config["key_name"]
  
  tags = var.tags
}

resource "aws_security_group" "web" {
  name_prefix = "web-sg"
  
  dynamic "ingress" {
    for_each = var.port_mappings
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
  
  tags = var.tags
}
```

### Demo 4: Variable with Object

In `variables.tf`:

```hcl
variable "database_config" {
  description = "Database configuration object"
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    allocated_storage = number
    db_name        = string
    username       = string
    backup_retention_period = number
    multi_az       = bool
    storage_encrypted = bool
  })
  default = {
    engine         = "mysql"
    engine_version = "8.0"
    instance_class = "db.t3.micro"
    allocated_storage = 20
    db_name        = "myapp"
    username       = "admin"
    backup_retention_period = 7
    multi_az       = false
    storage_encrypted = true
  }
}

variable "vpc_config" {
  description = "VPC configuration object"
  type = object({
    cidr_block           = string
    enable_dns_hostnames = bool
    enable_dns_support   = bool
    tags                 = map(string)
  })
  default = {
    cidr_block           = "10.0.0.0/16"
    enable_dns_hostnames = true
    enable_dns_support   = true
    tags = {
      Name = "main-vpc"
      Environment = "dev"
    }
  }
}

variable "application_config" {
  description = "Complete application configuration"
  type = object({
    name = string
    instances = object({
      count = number
      type  = string
      ami   = string
    })
    database = object({
      engine = string
      size   = string
    })
    networking = object({
      vpc_cidr = string
      subnets  = list(string)
    })
  })
  
  validation {
    condition     = var.application_config.instances.count > 0
    error_message = "Instance count must be greater than 0."
  }
}
```

Use it in `main.tf`:

```hcl
resource "aws_db_instance" "main" {
  identifier     = "${var.application_config.name}-db"
  engine         = var.database_config.engine
  engine_version = var.database_config.engine_version
  instance_class = var.database_config.instance_class
  allocated_storage = var.database_config.allocated_storage
  
  db_name  = var.database_config.db_name
  username = var.database_config.username
  password = "changeme123!"  # In production, use AWS Secrets Manager
  
  backup_retention_period = var.database_config.backup_retention_period
  multi_az               = var.database_config.multi_az
  storage_encrypted      = var.database_config.storage_encrypted
  
  skip_final_snapshot = true
  
  tags = {
    Name = "${var.application_config.name}-database"
  }
}

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_config.cidr_block
  enable_dns_hostnames = var.vpc_config.enable_dns_hostnames
  enable_dns_support   = var.vpc_config.enable_dns_support
  
  tags = var.vpc_config.tags
}
```

in terraform.tfvars

```hcl
application_config = {
  name = "webapp"
  instances = {
    count = 3
    type  = "t2.small"
    ami   = "ami-0c55b159cbfafe1d0"
  }
  database = {
    engine = "postgres"
    size   = "db.t3.small"
  }
  networking = {
    vpc_cidr = "10.0.0.0/16"
    subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  }
}

### Variable Validation

You can add validation rules to variables:

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
  
  validation {
    condition     = contains(["t2.micro", "t2.small", "t3.micro"], var.instance_type)
    error_message = "Instance type must be t2.micro, t2.small, or t3.micro."
  }
}
```

### Variable Input Methods

1. **Default values** in variable declarations
2. **Command line** with `-var` flag: `terraform apply -var="region=us-west-2"`
3. **Variable files** (.tfvars): `terraform apply -var-file="prod.tfvars"`
4. **Environment variables**: `TF_VAR_region=us-west-2 terraform apply`
5. **Interactive input** when running Terraform commands

## TF Dynamic Blocks

Dynamic blocks allow you to dynamically create nested blocks within resource configurations based on collections like lists or maps.

### Basic Syntax

```hcl
resource "aws_security_group" "example" {
  name        = "example"
  description = "Example security group"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      description = ingress.value.description
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

### Dynamic Block Example

In `variables.tf`:

```hcl
variable "ingress_rules" {
  description = "Ingress rules for security group"
  type = list(object({
    description = string
    port        = number
  }))
  default = [
    {
      description = "HTTP"
      port        = 80
    },
    {
      description = "HTTPS"
      port        = 443
    },
    {
      description = "SSH"
      port        = 22
    }
  ]
}
```

In `main.tf`:

```hcl
resource "aws_security_group" "example" {
  name        = "example"
  description = "Example security group"
  vpc_id      = aws_vpc.example.id
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      description = ingress.value.description
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## TF Resources

Resources are the most important element in Terraform. They represent infrastructure objects like virtual machines, networks, or DNS records.

### Resource Syntax:
```hcl
resource "resource_type" "resource_name" {
  argument = "value"
  # ...
}
```

### AWS Resource Examples:

#### VPC and Networking:
```hcl
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "main-vpc"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "main-igw"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "public-rt"
  }
}

resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${count.index + 1}"
  }
}

resource "aws_route_table_association" "public" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```

#### EC2 Instance with Security Group:
```hcl
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
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
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "web-security-group"
  }
}

resource "aws_key_pair" "main" {
  key_name   = "main-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

resource "aws_instance" "web" {
  count                  = 2
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t2.micro"
  key_name               = aws_key_pair.main.key_name
  vpc_security_group_ids = [aws_security_group.web.id]
  subnet_id              = aws_subnet.public[count.index].id
  
  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Web Server ${count.index + 1}</h1>" > /var/www/html/index.html
  EOF
  
  tags = {
    Name = "web-server-${count.index + 1}"
  }
}
```

#### Load Balancer:
```hcl
resource "aws_lb" "main" {
  name               = "main-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web.id]
  subnets            = aws_subnet.public[*].id
  
  enable_deletion_protection = false
  
  tags = {
    Name = "main-load-balancer"
  }
}

resource "aws_lb_target_group" "web" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }
}

resource "aws_lb_target_group_attachment" "web" {
  count            = length(aws_instance.web)
  target_group_arn = aws_lb_target_group.web.arn
  target_id        = aws_instance.web[count.index].id
  port             = 80
}

resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}
```

## TF Outputs

Outputs allow you to extract and display information about your infrastructure.

### outputs.tf
```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr_block" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "IDs of public subnets"
  value       = aws_subnet.public[*].id
}

output "web_instance_ips" {
  description = "Public IP addresses of web instances"
  value       = aws_instance.web[*].public_ip
}

output "load_balancer_dns" {
  description = "DNS name of the load balancer"
  value       = aws_lb.main.dns_name
}

output "database_endpoint" {
  description = "Database endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true  # Hide sensitive information
}

# Complex output with maps
output "instance_details" {
  description = "Detailed information about instances"
  value = {
    for i, instance in aws_instance.web : 
    "instance-${i}" => {
      id        = instance.id
      public_ip = instance.public_ip
      az        = instance.availability_zone
    }
  }
}

# Conditional output
output "environment_specific_info" {
  description = "Environment specific information"
  value = var.environment == "prod" ? {
    backup_enabled = true
    monitoring     = "enhanced"
  } : {
    backup_enabled = false
    monitoring     = "basic"
  }
}
```

### Accessing Outputs:
```bash
# View all outputs
terraform output

# View specific output
terraform output vpc_id

# Output in JSON format
terraform output -json

# Use output in other configurations
terraform output -raw load_balancer_dns
```

## TF Providers

### AWS Provider

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region     = var.aws_region
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
  
  # Alternative: Use AWS CLI profiles
  # profile = "default"
  
  default_tags {
    tags = {
      Terraform   = "true"
      Environment = var.environment
    }
  }
}

# Multiple provider configurations
provider "aws" {
  alias  = "us_east_1"
  region = "us-east-1"
}

provider "aws" {
  alias  = "us_west_2"
  region = "us-west-2"
}

# Use specific provider
resource "aws_instance" "east" {
  provider = aws.us_east_1
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t2.micro"
}
```

### GCP Provider

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
  credentials = file("path/to/service-account-key.json")
  project     = var.gcp_project_id
  region      = var.gcp_region
  zone        = var.gcp_zone
}

resource "google_compute_instance" "vm" {
  name         = "terraform-instance"
  machine_type = "e2-micro"
  zone         = var.gcp_zone
  
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  
  network_interface {
    network = "default"
    access_config {
      // Ephemeral public IP
    }
  }
}
```

### Proxmox Provider

```hcl
terraform {
  required_providers {
    proxmox = {
      source  = "telmate/proxmox"
      version = "2.9.14"
    }
  }
}

provider "proxmox" {
  pm_api_url      = "https://proxmox.example.com:8006/api2/json"
  pm_user         = "terraform@pve"
  pm_password     = var.proxmox_password
  pm_tls_insecure = true
}

resource "proxmox_vm_qemu" "vm" {
  name        = "terraform-vm"
  target_node = "proxmox-node"
  clone       = "ubuntu-20.04-template"
  
  cores   = 2
  sockets = 1
  memory  = 2048
  
  disk {
    size    = "20G"
    type    = "scsi"
    storage = "local-lvm"
  }
  
  network {
    model  = "virtio"
    bridge = "vmbr0"
  }
}
```

### TF Multiple Providers

```hcl
provider "aws" {
  region = "us-east-1"
  alias  = "east"
}

provider "aws" {
  region = "us-west-2"
  alias  = "west"
}

resource "aws_instance" "east_instance" {
  provider      = aws.east
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

resource "aws_instance" "west_instance" {
  provider      = aws.west
  ami           = "ami-0892d3c7ee96c0bf7"
  instance_type = "t2.micro"
}
```

## TF Modules

Modules are reusable Terraform configurations that help organize and standardize infrastructure code.

### Creating a Module

Create a directory structure:

```
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
main.tf
```

In `modules/vpc/variables.tf`:

```hcl
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}

variable "subnet_cidrs" {
  description = "CIDR blocks for subnets"
  type        = list(string)
}
```

In `modules/vpc/main.tf`:

```hcl
resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr
  tags = {
    Name = "module-vpc"
  }
}

resource "aws_subnet" "this" {
  count      = length(var.subnet_cidrs)
  vpc_id     = aws_vpc.this.id
  cidr_block = var.subnet_cidrs[count.index]
}
```

In `modules/vpc/outputs.tf`:

```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.this.id
}

output "subnet_ids" {
  description = "IDs of the subnets"
  value       = aws_subnet.this[*].id
}
```

### Using a Module

In your root `main.tf`:

```hcl
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr     = "10.0.0.0/16"
  subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  subnet_id     = module.vpc.subnet_ids[0]
}
```

### Using Public Modules

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.14.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  single_nat_gateway = true
}
```

## TF State Management

Terraform state is a critical component that maps real-world resources to your configuration.

### Local State

By default, Terraform stores state locally in a file named `terraform.tfstate`.

### Remote State

For team environments, it's better to use remote state storage:

```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### State Commands

```bash
# List resources in state
terraform state list

# Show details of a resource
terraform state show aws_instance.example

# Move a resource within state
terraform state mv aws_instance.old aws_instance.new

# Remove a resource from state
terraform state rm aws_instance.example

# Import existing infrastructure into state
terraform import aws_instance.example i-1234567890abcdef0
```

## TF Workspaces

Workspaces allow you to manage multiple states with the same configuration.

```bash
# List workspaces
terraform workspace list

# Create a new workspace
terraform workspace new dev

# Switch workspace
terraform workspace select prod

# Delete workspace
terraform workspace delete dev
```

Use workspace in configuration:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = terraform.workspace == "prod" ? "t2.medium" : "t2.micro"
  tags = {
    Environment = terraform.workspace
  }
}
```

## TF Functions

Terraform provides various built-in functions for string manipulation, numeric operations, collections, and more.

### String Functions

```hcl
local {
  upper_name = upper("example")         # EXAMPLE
  lower_name = lower("EXAMPLE")         # example
  title_name = title("hello world")     # Hello World
  joined     = join("-", ["a", "b", "c"]) # a-b-c
  substr     = substr("hello", 1, 3)     # ell
}
```

### Numeric Functions

```hcl
local {
  max_value = max(5, 12, 9)     # 12
  min_value = min(5, 12, 9)     # 5
  ceil_value = ceil(10.1)       # 11
  floor_value = floor(10.9)     # 10
}
```

### Collection Functions

```hcl
local {
  list_length = length(["a", "b", "c"])                # 3
  merged_map  = merge({a="1"}, {b="2"})              # {a="1", b="2"}
  map_keys    = keys({a="1", b="2"})                 # ["a", "b"]
  map_values  = values({a="1", b="2"})               # ["1", "2"]
  contains    = contains(["a", "b", "c"], "b")        # true
  element     = element(["a", "b", "c"], 1)           # "b"
  lookup      = lookup({a="1", b="2"}, "c", "default") # "default"
}
```

### Conditional Expressions

```hcl
local {
  instance_type = var.environment == "prod" ? "t2.medium" : "t2.micro"
}
```

## TF Provisioners

Provisioners let you execute actions on local or remote machines as part of resource creation or destruction.

### Local Exec Provisioner

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  provisioner "local-exec" {
    command = "echo 'Instance ${self.id} created with IP ${self.private_ip}' >> instance_info.txt"
  }
}
```

### Remote Exec Provisioner

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  key_name      = "example-key"
  
  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/example-key.pem")
    host        = self.public_ip
  }
  
  provisioner "remote-exec" {
    inline = [
      "sudo yum update -y",
      "sudo yum install -y httpd",
      "sudo systemctl start httpd",
      "sudo systemctl enable httpd"
    ]
  }
}
```

## TF Data Sources

Data sources allow you to fetch information about existing infrastructure that wasn't created by your current Terraform configuration.

```hcl
# Get latest Amazon Linux AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Get default VPC
data "aws_vpc" "default" {
  default = true
}

# Get availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Get existing security group
data "aws_security_group" "existing" {
  filter {
    name   = "group-name"
    values = ["existing-sg"]
  }
}

# Use data sources in resources
resource "aws_instance" "example" {
  ami               = data.aws_ami.amazon_linux.id
  instance_type     = "t2.micro"
  availability_zone = data.aws_availability_zones.available.names[0]
  vpc_security_group_ids = [data.aws_security_group.existing.id]
}
```

## Terraform Best Practices

### Version Constraints

```hcl
terraform {
  required_version = ">= 1.0.0, < 2.0.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}
```

### Sensitive Data Handling

```hcl
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}
```

### Tagging Strategy

```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    Owner       = "DevOps Team"
    ManagedBy   = "Terraform"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags          = merge(local.common_tags, { Name = "example-instance" })
}
```

## Advanced TF Features

### Count and For Each

```hcl
# Using count
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "server-${count.index}"
  }
}

# Using for_each with a map
resource "aws_instance" "server" {
  for_each = {
    web  = "t2.micro"
    app  = "t2.small"
    db   = "t2.medium"
  }
  
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value
  tags = {
    Name = "${each.key}-server"
  }
}

# Using for_each with a set
resource "aws_instance" "server" {
  for_each = toset(["web", "app", "db"])
  
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "${each.key}-server"
  }
}
```

### TF Expressions

```hcl
locals {
  # Conditional expression
  instance_type = var.environment == "prod" ? "t2.medium" : "t2.micro"
  
  # For expression
  upper_names = [for name in var.names : upper(name)]
  
  # For expression with map
  name_map = {for idx, name in var.names : idx => upper(name)}
  
  # For expression with filtering
  long_names = [for name in var.names : name if length(name) > 5]
}
```

### TF Lifecycle

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = false
    ignore_changes        = [tags]
  }
}
```

### TF Import

Import existing infrastructure into Terraform:

```bash
terraform import aws_instance.example i-1234567890abcdef0
```

### TF Modules with Versioning

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.14.0"
  
  # Module parameters
}
```

## TF Cloud and Enterprise Features

### Remote Backends

```hcl
terraform {
  backend "remote" {
    organization = "example-org"
    
    workspaces {
      name = "example-workspace"
    }
  }
}
```

### Sentinel Policies

Sentinel is a policy as code framework integrated with Terraform Cloud/Enterprise:

```hcl
# Example Sentinel policy
import "tfplan"

# Require specific instance types
allowed_types = ["t2.micro", "t2.small", "t3.micro", "t3.small"]

ec2_instances = filter tfplan.resource_changes as _, rc {
  rc.type is "aws_instance" and
  (rc.change.actions contains "create" or rc.change.actions contains "update")
}

violations = filter ec2_instances as _, instance {
  not (instance.change.after.instance_type in allowed_types)
}

main = rule {
  length(violations) is 0
}
```

## TF with CI/CD

### GitHub Actions Example

```yaml
name: Terraform

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
      with:
        terraform_version: 1.0.0
    
    - name: Terraform Init
      run: terraform init
    
    - name: Terraform Format
      run: terraform fmt -check
    
    - name: Terraform Validate
      run: terraform validate
    
    - name: Terraform Plan
      run: terraform plan
      if: github.event_name == 'pull_request'
    
    - name: Terraform Apply
      run: terraform apply -auto-approve
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
```

## Troubleshooting TF

### Common Issues and Solutions

1. **State Lock Issues**
   ```bash
   # Force unlock state
   terraform force-unlock LOCK_ID
   ```

2. **Debugging**
   ```bash
   # Enable detailed logs
   export TF_LOG=DEBUG
   export TF_LOG_PATH=terraform.log
   ```

3. **Plan File**
   ```bash
   # Save plan to file
   terraform plan -out=tfplan
   
   # Apply plan file
   terraform apply tfplan
   ```

4. **Refresh State**
   ```bash
   # Refresh state without making changes
   terraform refresh
   ```

## Conclusion

Terraform is a powerful tool for managing infrastructure as code across multiple providers. This guide covered the basics to advanced concepts, but there's always more to learn. As you become more comfortable with Terraform, explore the official documentation and community resources to deepen your knowledge.

Remember these key principles:

1. **Infrastructure as Code**: Treat your infrastructure like software
2. **Declarative Approach**: Define what you want, not how to get there
3. **State Management**: Understand and properly manage Terraform state
4. **Modularity**: Create reusable components
5. **Version Control**: Keep your Terraform code in version control


### Terraform Resources

- Handy CLI reference and syntax guide for quick lookup: [Terraform Cheat Sheet]({% post_url 2025-03-29-tfcheat %}){:target="_blank"}
- Official documentation covering syntax, modules, commands, and use cases: [Terraform Documentation](https://developer.hashicorp.com/terraform/docs){:target="_blank"}
- Detailed documentation for working with AWS resources using Terraform: [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs){:target="_blank"}
- A curated list of high-quality Terraform tools, modules, and tutorials: [Awesome Terraform (GitHub)](https://github.com/shuaibiyy/awesome-terraform){:target="_blank"}
- Manage remote state securely and collaborate on infrastructure-as-code projects: [Terraform Cloud (Free Tier)](https://app.terraform.io/signup){:target="_blank"}
- A static analysis security scanner for your Terraform code to detect misconfigurations: [TFSEC - Terraform Security Scanner](https://github.com/aquasecurity/tfsec){:target="_blank"}

Happy Terraforming!