---
author: [muju]
title: "Duplicati"
description: "A comprehensive guide to setting up Duplicati for backups using Docker, including core features, prerequisites, and configuration steps."
categories: ["Docker"]
image: 
  path: /assets/docker/duplicati/duplicati.webp
  lqip: data:image/webp;base64,UklGRmQBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSGgAAAABcFpt27I8v8bv/P3f4OuM4UyhW9DYzC2ROFTr7q8hA0TEBIB2Ge9A+8VduvDE4C7vEzfFlbhr/1BerFRUtcPdcH033NwNbw3dvlOAbgUYLAHcVD0AA8FhMpkMuikAv2A2dUhdRoYQAlZQOCDWAAAAEAYAnQEqFAAUAD6ROpdHpaMiITAIALASCWwAeDYQV4HAA24Ap9vF3FwEJlFmDKCZtokwqaaQfigAAP7D/v6p3xJ4PUY+ppFLQGLaoQ20lpad9JfhJqTSHbYCzYIWwXYSvkZuZRGNyyuBvJSvq0UI5k901lR8bUFVMVcq/4y0KPJbHcpg5KICoTirWJMD6LO+pyOa7NVRuNQTqE7gggV/Q8V2rPJqWiP/dDlDDjJMWchUfcOEE/t9QhP75A9YgixzMcYlohclHQjjQ04v+ZChah9P6AAAAA==
#toc: false
---

Duplicati is a powerful backup solution that supports standard protocols like FTP, SSH, WebDAV, and integrates with popular cloud services such as Microsoft OneDrive, Amazon Cloud Drive & S3, Google Drive, Box.com, Mega, hubiC, and more.
What is Duplicati?

Duplicati is an open-source backup software designed for secure and efficient backups. It supports strong encryption, incremental backups, and is highly configurable, making it suitable for both personal and professional use.
Core Features of Duplicati

- Incremental Backups: Save time and storage space by only backing up changes.
- Encryption: Protect your data with AES-256 encryption.
- Scheduling: Automate your backup tasks.
- Cross-Platform: Works on Windows, macOS, and Linux.
- Cloud Integration: Seamless integration with major cloud storage providers.

Why Use Duplicati?

Duplicati is ideal for users who need a flexible and reliable backup solution that can be tailored to different environments. Whether you're backing up data locally or to the cloud, Duplicati ensures your files are secure and easily recoverable.

## Prerequisites

1. [Docker]({% post_url 2023-01-15-docker %}){:target="_blank"} Engine and [Docker]({% post_url 2023-01-15-docker %}){:target="_blank"} Compose packages are installed and running
2. A non-root user with [Docker]({% post_url 2023-01-15-docker %}){:target="_blank"} privileges

## Setting Up the Duplicati Server

### Create the Directory Structure

To keep things organized, create a dedicated directory for Duplicati. Here's the file structure:

```bash
mkdir ~/docker/duplicati/data
touch ~/docker/duplicati/docker-compose.yml
```

```
📂 duplicati
|--📂 data
|   |--📂 config
|   |   |-- CDATXERRIL.backup
|   |   |-- Duplicati-server.sqlite
|   |   |-- GSZRNYUBKA.sqlite
|   |   |-- KYNVCKAOHG.sqlite
|   |   `-- control_dir_v2
|   |       `-- lock_v2
|   `--📂 scripts
|       `-- duplicati-after.sh
`--📑 docker-compose.yml

```

### Docker Configuration

With the directory structure in place, let’s edit the docker-compose.yml file using your favorite text editor. Here's an example configuration:

```yaml
services:
  duplicati-notifications:
    image: jameslloyd/duplicati-notifications
    container_name: duplicati-notifications
    restart: unless-stopped
    expose:
      - 5000
    networks:
      routevlan:
        ipv4_address: $IP
      proxy:

  duplicati:
    image: lscr.io/linuxserver/duplicati:latest
    container_name: duplicati
    environment:
      - TZ=Asia/Kolkata
      # - CLI_ARGS= #optional
      - PUID=1000
      - PGID=1000      
    volumes:
      - ./data/config:/config
      - ./data/scripts:/tmp/scripts
      - /root/docker:/docker
      - /root/script:/script
    ports:
      - 8200:8200
    restart: unless-stopped
    networks:
      routevlan:
        ipv4_address: $IP 
      proxy:
# Traefik labels can be uncommented and configured if using Traefik as a reverse proxy.
#    labels:
#      - "traefik.enable=true"
#      - "traefik.http.routers.duplicati.entrypoints=http"
#      - "traefik.http.routers.duplicati.rule=Host(`test.domain.tld`)"
#      - "traefik.http.middlewares.duplicati-https-redirect.redirectscheme.scheme=https"
#      - "traefik.http.routers.duplicati.middlewares=duplicati-https-redirect"
#      - "traefik.http.routers.duplicati-secure.entrypoints=https"
#      - "traefik.http.routers.duplicati-secure.rule=Host(`test.domain.tld`)"
#      - "traefik.http.routers.duplicati-secure.tls=true"
#      - "traefik.http.routers.duplicati-secure.service=duplicati"
#      - "traefik.http.services.duplicati.loadbalancer.server.port=8200"
#      - "traefik.docker.network=proxy"
#      - "traefik.http.routers.duplicati-secure.middlewares=authelia@file"
#      - "com.centurylinklabs.watchtower.enable=true"

networks:
  routevlan:
    external: true
  proxy:
    external: true
```

## Verify and Launch the Container

to verify the docker compose files has no issues use `docker compose config` Once you have the `docker-compose.yml` file you can run `docker-compose up -d` which will create the volume, download the image and start the container in the background 

To verify the status of all running containers, run the docker-compose command:

```bash
docker-compose ps (OR) docker ps -a
```

If something goes wrong, you can check the logs for each container using the docker logs command and specify the service name. For example, to check the log of the Nginx Proxy Manager container, run the following command:

```bash
docker logs $container (OR) docker compose logs -f
```

## Accessing the Duplicati Dashboard

If all is well, you can locally view your duplicati Server by navigating to http://$IP:8200

![duplicati](/assets/docker/duplicati/duplicati1.png)

## Creating a Backup Configuration

In this section, we'll create a backup for the Docker directory, using SFTP to store the backups on another server.

### Configure Backup Settings

Open the Duplicati web interface by navigating to http://$IP:8200. Click on Add backup to start the configuration process.

First, give your backup a name that helps you identify it easily, such as "Docker Directory Backup." If you want to encrypt your backup, select the encryption option and securely store the passphrase.

![duplicati](/assets/docker/duplicati/test.png){: width="800" height="500" } 

### Configure Backup Destination

For the storage type, select SFTP since you are backing up to another server. Enter the IP address or hostname of the remote server and specify the directory where you want the backups to be stored (e.g., /home/$user/backup). Provide the SFTP username and password or SSH key for the remote server. This setup ensures that your backups are securely transferred to and stored on the remote server.

![duplicati](/assets/docker/duplicati/test2.png){: width="800" height="500" } 

{: .prompt-info }
> Make sure you use the full path on the server (e.g., /home/$user/backup).

Next, navigate through the file browser in Duplicati to select the data you wish to back up. In this example, select the Docker directory. Ensure you use the full path to the directory (e.g., /home/$user/docker). You can include or exclude specific files and folders based on your needs.

![duplicati](/assets/docker/duplicati/test3.png){: width="800" height="500" } 

### Schedule the Backup

Configure the backup schedule to automate the process according to your needs. You might schedule daily or weekly backups, depending on how often your data changes. Additionally, set [retention policies](https://forum.duplicati.com/t/custom-backup-retention-testing/16250/2){:target="_blank"} to keep only a certain number of backups or delete backups older than a specified time.

![duplicati](/assets/docker/duplicati/test4.png){: width="800" height="500" } 

### Enable Notifications

While Duplicati doesn’t have built-in support for sending notifications to Discord, you can configure the duplicati-notifications container to send notifications upon backup completion. In Duplicati settings, you can configure email notifications or use scripts to trigger custom notifications. Set up duplicati-notifications to integrate with your Discord server, ensuring that you receive a detailed notification about the backup status in your Discord channel.

![duplicati](/assets/docker/duplicati/test5.png){: width="800" height="500" } || ![duplicati](/assets/docker/duplicati/test6.png){: width="800" height="500" } 

### Run the Backup

Once everything is configured, run the backup to ensure it’s working as expected. Monitor the progress in the Duplicati dashboard and view logs for any issues. After completion, a notification will be sent to your Discord server

![duplicati](/assets/docker/duplicati/test7.png){: width="800" height="500" } || ![duplicati](/assets/docker/duplicati/test8.png){: width="800" height="500" } 

### Verify and Restore

Periodically, it's a good practice to verify that your backups are functioning correctly and that you can restore data when needed. Duplicati provides a straightforward restore process, which can be initiated from the dashboard.

![duplicati](/assets/docker/duplicati/test9.png){: width="800" height="500" }

For more advanced configuration options, refer to the [Duplicati manual.](https://docs.duplicati.com/en/latest/){:target="_blank"}
