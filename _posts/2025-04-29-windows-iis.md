---
author: muju
title: "Windows Server 2025 on VMware Pro 🖥️"
description: "Step-by-step guide to installing the official Windows Server 2025 release on VMware Workstation Pro, including requirements, VM setup, and installation walkthrough."
categories: ["Windows"]
tags: ["VMware", "Windows Server", "SysAdmin", "Virtualization"]
image:
  path: /assets/windows/iis/inst/logo.png
  lqip: data:image/webp;base64,UklGRjYBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSH4AAAABcFtr29J8ULnbGDCAswCUHBbgpMsOoYpbm/SpkyqVu/c2QyZwqegjQm3bNgw9pR4CKQ51OnWOp6EVajFS2FQ+MPf75Ha/+lkjZzZB+M9Pvgpyt2uPd7038pinPm23rdG2+wZU0v9fAMCA/EvF8AX6/ehwGjiRnvE2qwBSHABWUDggkgAAAFAEAJ0BKhQAFAA+kUKcSiWjoqGoCACwEglmADFAL4A8qr2NgFAAcgpROwAAnfxqyyut63nMtVxRJQLd7h9ae2qx0905IhdvB9iCiFvex1a3vR59pQcKdf1L9w6v/fGqsIFHLAQaY9f5O4gGr+GywuumxXhfpGkZH8WTjml1AfD6I/OKf1JOMgh+PnTh9C6GyAAA
published: true
hidden: false
toc: true
---

## Introduction

Windows Server 2025 is the latest long-term servicing release (LTS) from Microsoft, bringing enhancements in performance, security, and hybrid cloud integration. Now that it’s officially released, system administrators and IT professionals can begin deploying it in test or production environments. This guide walks you through installing Windows Server 2025 on VMware Workstation Pro, covering prerequisites, VM setup, and step-by-step installation.

## What is Windows Server?

Windows Server is Microsoft's operating system designed for enterprise environments. It enables centralized management, network services, virtualization, identity services, and application hosting. Each version builds upon the last, providing improved features for performance, scalability, and security.

Some key features of Windows Server include:
- Active Directory & Group Policy Management
- Hyper-V Virtualization
- File and Storage Services
- Windows Admin Center
- DNS/DHCP and Network Policy Management

## What’s New in Windows Server 2025?

Windows Server 2025 includes many enhancements over Server 2022, including:

- Hotpatching for Desktop Experience – Apply updates without rebooting.
- SMB over QUIC – Secure file sharing without VPN.
- Enhanced Active Directory – Improved replication and logging.
- Modern Hardware Support – Support for newer CPUs and TPM 2.0.
- Improved Azure Arc Integration – Easier hybrid management.

## Requirements

Before proceeding, make sure you have the following 

> For a more detailed list of requirements, please refer to the official documentation [here](https://learn.microsoft.com/en-us/windows-server/get-started/hardware-requirements?tabs=cpu&pivots=windows-server-2025#components){:target="_blank"}
{: .prompt-info }

|Component | Minimum Requirement|
|----------|--------------------|
|RAM | 4 GB (8 GB+ recommended)|
|CPU | 1.4 GHz 64-bit processor (2 cores+)|
|Storage | 60 GB minimum|
|Virtualization | VMware Workstation 16 Pro or newer|
|ISO File | Official Windows Server 2025 ISO |

You can download the ISO from Microsoft’s official website from [here](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2025?msockid=1dbd6c8bbd85672a1985788bbc2d66b4){:target="_blank"}

## Create a New Virtual Machine

- Open VMware Workstation Pro.
- Click Create a New Virtual Machine.
- Choose Typical (recommended) and click Next.
- Select Installer disc image file (iso) and browse to the Windows Server 2025 ISO.
- Choose the Guest OS version as Windows Server 2022 x64 (until VMware updates to list 2025).
- Set a name and location for your VM.
- Allocate at least 4 GB RAM and 2 CPUs.
- Create a virtual disk (recommended: 60 GB or more).
- Finish setup and power on the VM.

## Start the Installation

- Press any key to start the installation.

![IIS](/assets/windows/iis/inst/iis0.png){: width="600" height="100" } 

- Select the Language Time and Currency Format.

![IIS](/assets/windows/iis/inst/iis2.png){: width="600" height="100" }

- Select the Keyboard setting.

![IIS](/assets/windows/iis/inst/iis3.png){: width="600" height="100" }

- Select 'Install Windows Server' and select the checkbox to agree to delete files, apps, and settings.

![IIS](/assets/windows/iis/inst/iis4.png){: width="600" height="100" }

- Select 'Install Windows Server' and check the box to agree to delete files, apps, and settings.
- For demo purposes, select 'I don’t have a key.

![IIS](/assets/windows/iis/inst/iis5.png){: width="600" height="100" }

- Select The Image or choose the server edition to install

![IIS](/assets/windows/iis/inst/iis6.png){: width="600" height="100" }

- The system is Ready to install the Windows Server 2025 Datacenter (Desktop Experience)

![IIS](/assets/windows/iis/inst/iis7.png){: width="600" height="100" }

- Windows Server 2025 was successfully installed.

![IIS](/assets/windows/iis/inst/iis8.png){: width="600" height="100" }


## Internet Information Services (IIS)


**Step 1.** Open the Server Manager.

**Step 2.** Click on "Manage".

**Step 3.** Then select "Add Roles and Features".

![IIS](/assets/windows/iis/inst/iis9.png){: width="600" height="100" }

**Step 4.** On the "Before you begin" screen, click "Next."

![IIS](/assets/windows/iis/inst/iis10.png){: width="600" height="100" }

**Step 5.** Choose "Role-based or feature-based installation"

**Step 6.** And then click "Next."

![IIS](/assets/windows/iis/inst/iis11.png){: width="600" height="100" }

**Step 7.** Choose the appropriate server from the server pool.

**Step 8.** And then click "Next".

![IIS](/assets/windows/iis/inst/iis12.png){: width="600" height="100" }

**Step 9.** Select the server role "Web Server (IIS)."

**Step 10.** If a box appears asking to add required features, click "Add Features."

**Step 11.** And then click "Next."

![IIS](/assets/windows/iis/inst/iis13.png){: width="600" height="100" }

**Step 12.** Select the features screen, you can choose extra features if you need them, and then Click "Next".

![IIS](/assets/windows/iis/inst/iis14.png){: width="600" height="100" }

**Step 13.** Web Server Role (IIS)" screen, review the information about IIS and click "Next".

![IIS](/assets/windows/iis/inst/iis15.png){: width="600" height="100" }

**Step 14.** Select role services, anything you want to install, and Make sure to select important services.

When setting up the IIS (Internet Information Services) web server role, there are some important services you need to choose to make sure the web server works properly.

  | **Category**             | **Features**                                                                 |
  |--------------------------|------------------------------------------------------------------------------|
  | **Web Server**           | • Static Content  <br>• Default Document  <br>• Directory Browsing  <br>• HTTP Errors  <br>• HTTP Redirection (if needed) |
  | **Application Development** | • ASP.NET (for .NET applications)  <br>• .NET Extensibility  <br>• ISAPI Extensions  <br>• ISAPI Filters |
  | **Management Tools**     | • IIS Management Console  <br>• IIS Management Scripts and Tools  <br>• Management Service (for remote management) |
  | **Performance**          | • Static Content Compression  <br>• Dynamic Content Compression              |
  | **Security**             | • Request Filtering  <br>• Windows Authentication (if needed)  <br>• Basic Authentication (if needed)  <br>• URL Authorization |

**Step 15.** Then Click "Next"

![IIS](/assets/windows/iis/inst/iis16.png){: width="600" height="100" }

**Step 16.** Review and click "Install".

![IIS](/assets/windows/iis/inst/iis17.png){: width="600" height="100" }

**Step 17.** After installation is finished, click "Close".

![IIS](/assets/windows/iis/inst/iis18.png){: width="600" height="100" }

## Verify the IIS server role Installation

1.  open a web browser and enter [http://IP](http://localhost/){:target="_blank"}
2.  confirming that IIS is running.  
    ![IIS](/assets/windows/iis/inst/iis19.png){: width="600" height="100" }


## Conclusion

Installing Windows Server 2025 on VMware Workstation Pro is a great way to test or build out lab environments. Whether you’re preparing for production or exploring new features, this setup gives you a solid foundation to begin working with Microsoft’s latest server OS. With its improved hybrid capabilities and system enhancements, Windows Server 2025 is ready to power modern infrastructure.
