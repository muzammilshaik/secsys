---
author: muju
title: "Jenkins Rsync Backup & Discord Alerts"
description: "This guide provides step-by-step instructions to configure the Periodic Backup plugin in Jenkins for automated backups."
categories: ["Linux"]
tags: ["Linux", "Jenkins"]
image:
  path: /assets/vm/ct/jenkins/j.webp
  lqip: data:image/webp;base64,UklGRu4BAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSJ0AAAABgJpt27Lsxp1GYghI1iC6LMEALtndyURrHEzABCT53d3df/x93xEiYgKwVuntn31/7rU02MyLv8zW//g2cOszUsc6y4y4u8Z0SnZrBGD5mlG+m6F+mFHfqKozhqljFpM/Fu+PLC76LNqmf7o/HYp0W1JwS1SzMIAcVQ+A64ngtxkb9LQAIPY0h2t+xyCWP678HYuIEL36+XsbcbEKAFZQOCAqAQAAkAYAnQEqFAAUAD6RQJhJpaOiISgKqLASCWIAUwZN23wLJcIhjdjBr6VyUcPvXfMV3IUqkDGuSED0Gn15QAD+wgdremj5uMHLQ8/JZ3QDRler5lCxn2HSLL2YFDKDXPr08Lo+rLQtkRIazXnr+6ZS6zM6T/79ZTLj/NI3PZ7MkB8Oa5ysX6+WNjVqnXGtKU0q6fOvdPT0VTyDlzbv/zB6lHRg2ZHYg8d/iZwbeaxSzITYC3fxU8rnm2mJ8monz9Z+Wy9LEQCPBrTCfCiTSzJjbzPy3/fZuEvQaMADD+FoKY1TBSuX98WrKKWJ5kh4nQz45I8p0Rb9AmPDiJ8aHo6F2foZBVda3tzm/2fOUzj481foUt7x2xavkoVmVSz9gs0l3fJgKuwlkAAAAA==
published: true
hidden: false
toc: true  # Table of Content
---

## **Prerequisites**
1. Ensure Jenkins is installed and running.
2. Install the "Periodic Backup" plugin:
   - Go to **Manage Jenkins > Plugin Manager > Available Plugins**.
   - Search for "Periodic Backup" and install it.
   - Restart Jenkins after installation.

---

## ⚙️ **Configuration Steps**

### 1️⃣ **Access the Periodic Backup Manager**
- Navigate to **Go to Jenkins Dashboard → Manage Jenkins**
- Scroll down to **Periodic Backup Manager**.

### 2️⃣ **Configure Basic Settings**
Click on the **Configure** button within the Periodic Backup Manager and set the following parameters:

#### 📂 **Temporary Directory**
- Specify a path for creating archives during backup, restoring archives, and unpacking their content. EG: `/var/lib/jenkins/tmp`
- Ensure:
  - The directory is writable and empty.
  - It is located outside the Jenkins home directory to avoid data loss.

#### 🕒 **Backup Schedule (Cron)**
- Define when backups should occur using cron syntax.
  - Example: `H 2 * * *` (Runs at 2 AM daily).
- Use the **Validate Cron** button to verify your schedule.

#### 🗑️ **Maximum Backups in Location**
- Set the maximum number of backups to retain in each location. `EG: 7`
- Older backups exceeding this limit will be deleted automatically.

#### 📅 **Store No Older Than (Days)**
- Specify the retention period for backups in days.
- Backups older than this value will be removed.

---

### 3️⃣ **File Management Strategy**
Choose a file management strategy:
- Select a suitable **FileManager** 
- Fill in required fields as per your choice.

---

### 4️⃣ **Storage Strategy**
- storage strategies will help us to compress the data yusing you favourire compressions (eg: TarGzStorage ).
- Fill in necessary details for each storage type.

#### 📍 **Backup Location**
- Specify where backups should be stored (e.g., `/var/backups/jenkins`).
- Use the **Validate Path** button to verify your directory has write access and ☑️ Enable this location
- Avoid using paths inside the temporary directory to prevent data conflicts.

---

### 💾 **Save Configuration**
- Review all settings and ensure they are correct.
- Click **Save** to apply changes.

---

## 🔍 **Testing the Configuration**
1. Go back to the Periodic Backup Manager.
2. Click on **Backup Now** to initiate a manual backup.
3. Verify that a backup is created in the specified location.

![jenkins](/assets/vm/ct/jenkins/backup/backup1.png){: width="800" height="500" }

---

## ♻️ **Restoring from Backup**
1. Navigate to the Periodic Backup Manager.
2. Select the desired backup from the list of available backups.
3. Click on **Restore** and follow on-screen instructions.

---

## 🌐 **Upload Backup to SFTP**

### 1️⃣ Install the "Publish Over SSH" Plugin
- Open Jenkins Dashboard.
- Navigate to Manage Jenkins > Manage Plugins.
- In the Available tab, search for SSH Agent Plugin.
- Select it and click Install without Restart.
- Add the SFTP Credentials

### 2️⃣ Create a Jenkins Free-Style Project

- Go to Jenkins Dashboard > New Item.
- Select Free-Style Project, name it as `Backup SFTP` and Click OK.

### 3️⃣ Configure the Build Steps

- Triggers Poll SCM `H/3 * * * *` the cron will schedule to run the job everyday at 3:00 AM
- Environment `SSH Agent` and select the credentials 
- Scroll to Build section and select the Execute shell and paste the script as this script is self explanatory it will send the notification to discord thread once the file synchronization is completed

> Please change the credentials, IP and username accordingly
{: .prompt-info }


```bash
send_notification() {
    local start="$1"
    local end="$2"
    local info="$3"

    # Escape special characters in the info string
    escaped_info=$(echo "$info" | sed 's/"/\\"/g' | sed ':a;N;$!ba;s/\n/\\n/g')

    curl -H "Content-Type: application/json" -X POST -d '{
        "embeds": [
            {
                "color": 966115,
                "description": "👨‍💻  **__Jenkins_Dietpi_Backup__** 🚀 \n\n **⏱️ Execution Time:** '"$start"' \n\n **️ 📝  [Access LOGS from here](https://yunohost.home.lab/jenkins/job/Dotfiles%20Auto/'"$BUILD_NUMBER"'/console)**"
            }
        ]
    }' "https://discord.com/api/webhooks/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}

###########################################
## Updating the server
###########################################
start=$(timedatectl | grep Local | awk '{print $5,$6}')

rsync -avz --progress -e "ssh -o StrictHostKeyChecking=no" /var/backups/jenkins/ $USER@$iP:/REMOTE/LOCATION/

# Sending the notification 
send_notification "$start"
```

Save the configuration.

![jenkins](/assets/vm/ct/jenkins/backup/backup2.png){: width="800" height="500" }
![jenkins](/assets/vm/ct/jenkins/backup/backup3.png){: width="800" height="500" } 

## 🌟 **Best Practices**
1. Regularly test your backups by restoring them on a test environment.
2. Use an external storage location (e.g., cloud or network drive or Remote server) for added security.
3. Monitor disk space usage in your backup location to avoid failures.

---

By following this guide, you can ensure that your Jenkins configurations and data are backed up periodically, safeguarding against potential data loss or system failures.
