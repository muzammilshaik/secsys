---
author: muju
title: "Terraform + Cloud-Init VM Deploy ☁️"
description: "Learn how to automate virtual machine provisioning using Terraform combined with Cloud-Init and create your own reusable cloud-init enabled template on Proxmox."
categories: ["Linux"]
tags: ["Automation", "Terraform", "LXC", "DevOps"]
image:
  path: /assets/dev/tf/tfvm/logo.png
  lqip: data:image/webp;base64,UklGRvwAAABXRUJQVlA4IPAAAACwBQCdASoUABQAPpE+m0kloyKhKAqosBIJaACxAQduwDeXCVKeMUhXpQgco0ORH8vV2lo3OPAA/vuki/pRfQKwu7H5X/zLg5/lp0tCJJhFO+nIvZ51NhJenTPjXv5tt5WNwiUKnOv4VFbiHK0ppzpgzGZW0XgAQXaqJ3d3OPCdHRbkLVUatWm+IrgrAq8zhxTFejtElwQCtgP5Ej7WOs8Q5P0CC5IxsoDxKegCKF4zhIGF3D93NgbhfMa23MNrhYHSfViVqb4TVnOSWbxxYTc3VG12l20KcuacqMm7GyrIKEKiRu/8preuW6JRAqAAAAA=
published: true
hidden: false
toc: true
---

Provisioning virtual machines manually every time can get repetitive and error-prone. By using Terraform to automate VM creation and Cloud-Init to configure the VM on boot, we can streamline the process for consistent infrastructure deployments.

In this blog post, we’ll walk through a simple but powerful setup for provisioning a virtual machine with Terraform while injecting a custom Cloud-Init configuration to handle OS-level setup.

## 🧰 Prerequisites

Make sure you have the following before starting:

- ✅ Terraform installed on your machine
- ✅ Access to a Proxmox server or similar virtualization platform
- ✅ A base Cloud-Init enabled template image
- ✅ SSH key pair generated
- ✅ Basic understanding of Terraform

## 📁 Directory Structure

I've added the complete Terraform code to my GitHub repository. You can easily clone it and start spinning up your Proxmox VMs or LXCs using Cloud-Init templates.

```bash
git clone https://github.com/s3csys/terraform.git
```

```bash
📁 terraform
├── 📄 LICENSE
├── 📄 README.md
├── 📁 proxmox_debian_example
├── 📁 proxmox_minikube_cloudinit_example
├── 📁 proxmox_multi-lxc_example
└── 📁 proxmox_vm_cloudinit_example
    ├── 📄 cloudinit.tf
    ├── 📁 files
    │   ├── 📄 cloudinit.yml
    │   └── 🔑 id_rsa
    ├── 📄 main.tf
    ├── 📄 provider.tf
    ├── 📄 terraform.tfvars
    └── 📄 vars.tf
```

## 📦 Step 1: Cloud  VM Template

Before Terraform can deploy VMs using Cloud-Init, you need a base template ready. Here’s how to create one using an Ubuntu Cloud image:

```bash
# 📥 Download the latest Ubuntu cloud image
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img

# 🛠️ Install libguestfs-tools for customizing the image
apt update -y && apt install libguestfs-tools -y

# 📦 Install qemu-guest-agent into the image
virt-customize -a noble-server-cloudimg-amd64.img --install qemu-guest-agent

# 🖥️ Create a new Proxmox VM
qm create 5000 --memory 2048 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci

# 💾 Import the disk to local-lvm (must use full path)
qm set 5000 --scsi0 local-lvm:0,import-from=/root/noble-server-cloudimg-amd64.img

# ⚙️ Configure Cloud-Init drive and serial console
qm set 5000 --ide2 local-lvm:cloudinit
qm set 5000 --serial0 socket --vga serial0

# 🧑‍💻 Ensure proper boot settings
qm set 5000 --boot c --bootdisk scsi0

# 🧰 Convert the VM into a reusable template
qm template 5000

```

Your Proxmox template is now ready to be cloned with Terraform

```bash
❯ virt-customize -a noble-server-cloudimg-amd64.img --install qemu-guest-agent
[   0.0] Examining the guest ...
[  40.7] Setting a random seed
virt-customize: warning: random seed could not be set for this type of
guest
[  40.8] Setting the machine ID in /etc/machine-id
[  40.8] Installing packages: qemu-guest-agent
[ 126.9] Finishing off

❯ qm create 5000 --memory 2048 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci

❯ qm set 5000 --scsi0 local-lvm:0,import-from=/tmp/template/noble-server-cloudimg-amd64.img
update VM 5000: -scsi0 local-lvm:0,import-from=/tmp/template/noble-server-cloudimg-amd64.img
  Logical volume "vm-5000-disk-0" created.
transferred 0.0 B of 3.5 GiB (0.00%)
transferred 35.8 MiB of 3.5 GiB (1.00%)
transferred 71.7 MiB of 3.5 GiB (2.00%)
transferred 3.5 GiB of 3.5 GiB (100.00%)
scsi0: successfully created disk 'local-lvm:vm-5000-disk-0,size=3584M'

❯ qm set 5000 --serial0 socket --vga serial0
update VM 5000: -serial0 socket -vga serial0

❯ qm set 5000 --boot c --bootdisk scsi0
update VM 5000: -boot c -bootdisk scsi0

❯ qm template 5000
  Renamed "vm-5000-disk-0" to "base-5000-disk-0" in volume group "pve"
  Logical volume pve/base-5000-disk-0 changed.
  WARNING: Combining activation change with other commands is not advised.
```

## ✍️ Step 2: Cloud-Init Configuration

Create a `cloudinit.yml` file which will handle initial setup inside the VM:

```yaml
#cloud-config
hostname: terraform-vm
fqdn: terraform-vm.local

users:
  - name: ${vm_user}
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    groups: sudo, docker
    shell: /bin/bash
    ssh-authorized-keys:
      - ${ssh_key}
    disable_root: false
    ssh_pwauth: false

package_update: true
package_upgrade: true
packages:
  - curl
  - htop

runcmd:
  - echo "Welcome to Terraform provisioned VM!" | sudo tee /etc/motd
```

> 💡 Tip: You can add more packages or commands to this file as needed for your environment.
{: .prompt-info }

## 🚀 Step 3: Terraform Provider (provider.tf)

To get started with automating VM deployments on Proxmox using Terraform, you need to configure the Proxmox provider with authentication variables and logging for debugging purposes. Below is the complete `providers.tf` setup using the Telmate/Proxmox provider.

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

> 💡 Make sure to define all referenced variables in your variables.tf or provide them through a terraform.tfvars file or environment variables.
{: .prompt-info }

## ⚙️ Step 4: Terraform Config (main.tf)

In Terraform `main.tf` is typically the primary configuration file where you define the main infrastructure resources you want to provision. The `.tf` stands for Terraform file, and while Terraform doesn't require any specific filename (as long as it ends in .tf), `main.tf` is a common convention used to keep things organized. `main.tf` provisions a Proxmox VM using Terraform by cloning from a template, enabling cloud-init for automation

```yaml
resource "proxmox_vm_qemu" "cloudinit-test" {
    desc        = "testing the terraform and cloudinit"
    name        = var.vm_name
    target_node = var.pm_node
    clone       = var.template_name     # The template name to clone this vm from
    full_clone  = true
    agent       = 1         # Activate QEMU agent for this VM
    os_type     = "cloud-init"
    cores       = 2
    sockets     = 1
    vcpus       = 0
    cpu_type    = "host"
    memory      = 2048
    scsihw      = "virtio-scsi-pci"

    # Setup the disk
    disks {
        ide {
            ide2 {
                cloudinit {
                    storage = var.storage_pool
                }
            }
        }
        scsi {
            scsi0 {
                disk {
                    size            = 10
                    cache           = "writeback"
                    storage         = var.storage_pool
                    #storage_type    = "rbd"
                    #iothread        = true
                    #discard         = true
                    replicate       = true
                }
            }
        }
    }

    network {
        id = 0
        model = "virtio"
        bridge = var.network_bridge
    }

    # Setup the ip address using cloud-init.
    boot = "order=scsi0"
    ipconfig0 = "ip=20.20.20.150/24,gw=20.20.20.1"
    #ipconfig0 = "ip=dhcp"
    serial {
      id   = 0
      type = "socket"
    }
    
    ciuser = "var.vm_user"
    cipassword = var.vm_password
    cicustom = "user=local:snippets/cloudinit-${var.vm_name}.yml"
}
```

> Note: Proxmox must already have your Cloud-Init user data file uploaded as a snippet. 
{: .prompt-info }

## 📦 Step 5: Upload Cloud-Init snippet

This `cloud-init.tf` script automates the process of rendering a dynamic cloud-init YAML file and securely uploading it to a Proxmox server. 

```yaml
# 📄 cloud-init.tf - Generate and Upload Cloud-Init Config using Terraform
# 🧠 Reads the cloud-init user-data file with dynamic variables
data "template_file" "cloudinit_userdata" {
  template = file("${path.module}/files/cloudinit.yml")

  vars = {
    hostname = var.vm_name
    vm_user  = var.vm_user 
    ssh_key  = var.vm_ssh_key
  }
}

# 📝 Writes the rendered user-data to a local file
resource "local_file" "rendered_cloudinit" {
  content  = data.template_file.cloudinit_userdata.rendered
  filename = "${path.module}/rendered_cloudinit.yml"
}

# 🚀 Uploads the file using scp after rendering
resource "null_resource" "scp_cloudinit" {
  depends_on = [local_file.rendered_cloudinit]

  provisioner "local-exec" {
    command = <<EOT
      scp -i ${path.module}/files/id_rsa \
          -o StrictHostKeyChecking=no \
          ${local_file.rendered_cloudinit.filename} \
          root@${var.proxmox_host}:/var/lib/vz/snippets/cloudinit-${var.vm_name}.yml
    EOT
  }
}
```

## 🧮 Step 6: Variables and Values (variables.tf and terraform.tfvars)

The providers file in Terraform is crucial for defining which external systems or platforms Terraform should interact with. In this case, the configuration specifies the Proxmox provider, which enables Terraform to communicate with a Proxmox VE environment for automating virtual machine provisioning.

At the top, the terraform block declares the required provider and its version (Telmate/proxmox at version 3.0.1-rc6). This ensures that the appropriate plugin is downloaded and used during Terraform operations.

```yaml
# 🔐 Proxmox Authentication
variable "proxmox_pm_api_url"       { type = string }
variable "proxmox_PM_USER"          { type = string }
variable "proxmox_PM_API_TOKEN_ID"  { type = string, sensitive = true }
variable "proxmox_PM_API_TOKEN_SECRET" { type = string, sensitive = true }

# 💻 VM Configuration
variable "vm_name"     { type = string }
variable "vm_user"     { type = string }
variable "vm_password" { type = string, sensitive = true }
variable "vm_ssh_key"  { type = string }

# 🖥️ Proxmox VM Infrastructure
variable "pm_node"        { type = string }
variable "template_name"  { type = string }
variable "storage_pool"   { type = string }
variable "network_bridge" { type = string }
variable "proxmox_host"   { type = string }
```

```yaml
# 🔐 Proxmox Authentication
proxmox_pm_api_url         = "https://<PROXMOX_IP>:8006/api2/json"
proxmox_PM_USER            = "terraform@pve"
proxmox_PM_API_TOKEN_ID    = "terraform@pve!terraform"
proxmox_PM_API_TOKEN_SECRET = "ffb26b21-abcd-defg-hijkl-01feffc7e6e4"

# 💻 VM Configuration
vm_name      = "cloudinit"
vm_user      = "your_vm_user"
vm_password  = "your_secure_password"
vm_ssh_key   = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIaaaaaaaaaaaaaaa/qqa9Mtkzn6G3oaM03AFKinPjO @public_key"

# 🖥️ Proxmox VM Infrastructure
pm_node        = "pve"
template_name  = "<VM_TEMPLATE_ID>"  # e.g., "9000" or any existing template VM ID
storage_pool   = "local-lvm"
network_bridge = "vmbr0"
proxmox_host   = "<PROXMOX_SSH_IP>"  # Used for SCP upload (e.g., "192.168.1.10")
```

## 🚀 Step 7: Initialize & Apply

Once your Terraform configuration files and variables are set up, it’s time to deploy the infrastructure. This step involves running a series of Terraform commands to initialize the environment, review the execution plan, and apply the configuration to provision the virtual machine.

- Initialize the Working Directory
  ```bash
  terraform init
  ```
  
- Review the Execution Plan
  ```bash
  terraform plan --out="tfplan"
  ```

- Apply the Configuration
  ```bash
  terraform apply "tfplan"
  ```

Using Terraform + Cloud-Init + a custom Proxmox VM template, you can automate and scale your infrastructure with ease. This approach is perfect for repeatable, secure, and hands-off provisioning for test labs, production, or homelab environments.

Feel free to extend this setup by turning it into a module or integrating with CI/CD pipelines. For a quick reference on Terraform commands, check out the [Terraform Cheatsheet]({% post_url 2025-03-29-tfcheat %}){:target="_blank"}


