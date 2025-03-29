---
author: muju
title: "Terraform Proxmox LXC Containers"
description: "Learn how to provision and manage LXC containers using Terraform. This guide covers installation, configuration, and best practices."
categories: ["Linux"]
tags: ["Automation", "Terraform", "LXC", "DevOps"]
image:
  path: /assets/dev/tf/tf-logo1.webp
  lqip: data:image/webp;base64,UklGRjwBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSJYAAAABgCNJkmqnzY8uMzN/sWShrXczMTMzM1zr4752dl+sbhARE/A3BY3k16gO8rWlg+Zk1dKkhAviuqgY+aSCFGc5AM0XkottBTk4Ct6bJInQWldBzlMKILrd0wAkqgNLHQncxulA0d0Iy8LizLgSnGd/JwZlmBwcGXfThylYDJNkf9cGyxLS839+oQPFK6EBqNyZGoDGswJWUDgggAAAAFAEAJ0BKhQAFAA+kUCYSaWjoiEoCqiwEglsAHjSIG0dt4rTAN5uAzTitYAA2m//Yww3TGemeBdksKYdQkbXNM+CP+XeJRKiT9+lNUmPT/+UF2Ih7KrEQ9lVbc3cpxQwXFTa5Lbxsbdebl/PyU95JuX8/JO7V8a4EOn/Nd6AAAAA
published: true
hidden: false
toc: true
# ref: https://www.tecmint.com/install-terraform-in-linux/
# terraform@pve!terraform
# 4b3953d4-9d3c-4a01-9fae-cd79e22bdc24
---

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

![Terraform](/assets/dev/tf/tf-api.png){: width="800" height="500" }

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
    ip     = "20.20.20.150/24"
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

![Terraform](/assets/dev/tf/tf-var-verify.png){: width="800" height="500" }

🔥 Conclusion

By integrating Terraform with LXC, you can automate container provisioning efficiently. This approach enhances reproducibility, scalability, and version control for your infrastructure. Explore more Terraform configurations and enhance your containerized environment!

For a quick reference on Terraform commands, check out the [Terraform Cheatsheet]({% post_url 2025-03-29-tfcheat %}){:target="_blank"}