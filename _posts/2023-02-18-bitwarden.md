---
author: [muju]
title: "Bitwarden"
description: ""
categories: ["Docker"]
image: /assets/docker/vault/vaultwarden-light.png
#toc: false
---

Bitwarden is a free and open-source password management service that stores sensitive information such as website credentials in an encrypted vault. The Bitwarden platform offers a variety of client applications including a web interface, desktop applications, browser extensions, mobile apps, and a CLI.

I use Bitwarden as my main password vault. It stores my card details for automating the filling out of payment forms. Saves me from having to find or remember my card details. I also use Bitwarden for storing all of my passwords.

Having Bitwarden as a public endpoint means that I can connect to my password vault using the Bitwarden app on Android, specifying my self hosted instance.

## Prerequisites

1. [Docker]({% post_url 2023-01-15-docker %}){:target="_blank"} Engine and [Docker]({% post_url 2023-01-15-docker %}){:target="_blank"} Compose packages are installed and running
2. A non-root user with [Docker]({% post_url 2023-01-15-docker %}){:target="_blank"} privileges

## Setting up the Bitwarden Server

### Create the directory structure

To keep the docker directory clean i have created a bitwarden directory to store the configuration file. My file structure is as follows:

```bash
mkdir ~/docker/bitwarden/data
touch ~/docker/bitwarden/docker-compose.yml
```
```
📂bitwarden
|-- 📂data
|-- 📑docker-compose.yml  
```

### Docker Configuration

As we have created the docker compose file lets edit the files with your favorite editors of choice

```yaml
version: '3.8'

services:
  bitwarden:
    image: vaultwarden/server:latest
    container_name: bitwarden
    volumes:
      - ./data:/data
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - WEBSOCKET_ENABLED=true
      - SIGNUPS_ALLOWED=true
      - DOMAIN=https://subdomain.yourdomain.com
      - LOGIN_RATELIMIT_MAX_BURST=10 
      - LOGIN_RATELIMIT_SECONDS=60
      - ADMIN_RATELIMIT_MAX_BURST=10
      - ADMIN_RATELIMIT_SECONDS=60
      - ADMIN_TOKEN=YourReallyStrongAdminTokenHere
      - SENDS_ALLOWED=true
      - EMERGENCY_ACCESS_ALLOWED=true
      - WEB_VAULT_ENABLED=true

      - SIGNUPS_ALLOWED=false
      - SIGNUPS_VERIFY=true
      - SIGNUPS_VERIFY_RESEND_TIME=3600
      - SIGNUPS_VERIFY_RESEND_LIMIT=5
      - SIGNUPS_DOMAINS_WHITELIST=yourdomainhere.com,anotherdomain.com

      - SMTP_HOST=smtp.youremaildomain.com
      - SMTP_FROM=vaultwarden@youremaildomain.com
      - SMTP_FROM_NAME=Vaultwarden
      - SMTP_SECURITY=SECURITYMETHOD
      - SMTP_PORT=XXXX
      - SMTP_USERNAME=vaultwarden@youremaildomain.com
      - SMTP_PASSWORD=YourReallyStrongPasswordHere
      - SMTP_AUTH_MECHANISM="Mechanism"
    ports:
      - "8080:80"
#    networks:
#      - proxy
#
#networks:
#  proxy:
#    external: true
```

## Environment variables

### **System Settings**
- **Domain:** This is the domain you wish to associate with your Vaultwarden instance.
- **LOGIN_RATELIMIT_MAX_BURST:** This is the maximum number of requests allowed in a burst of login / two-factor attempts while maintaining the average specified in LOGIN_RATELIMIT_SECONDS.
- **LOGIN_RATELIMIT_SECONDS:** This is the average number of seconds between login requests from the same IP before Vaultwarden rate limits logins.
- **ADMIN_RATELIMIT_MAX_BURST:** This is the same as LOGIN_RATELIMIT_MAX_BURST, only for the admin panel.
- **ADMIN_RATELIMIT_SECONDS:** This is the same as LOGIN_RATELIMIT_SECONDS, only for the admin panel.
- **ADMIN_TOKEN:** This value is the token (a type of password) for the Vaultwarden admin panel. For security, this should be a long random string of characters. The admin panel is disabled if this value is not set.
- **SENDS_ALLOWED:** This setting determines whether users are allowed to create Bitwarden Sends – a form of credential sharing.
- **EMERGENCY_ACCESS_ALLOWED:** This setting controls whether users can enable emergency access to their accounts. This is useful, for example, so a spouse can access a password vault in the event of death so they can gain access to account credentials. Possible values: true / false.
- **WEB_VAULT_ENABLED:** This setting determines whether or not the web vault is accessible. Stopping your container then switching this value to false and restarting Vaultwarden could be useful once you’ve configured your accounts and clients to prevent unauthorized access. Possible values: true/false.

### **Signup Settings**
- **SIGNUPS_ALLOWED:** This setting controls whether or not new users can register for accounts without an invitation. Possible values: true / false.
- **SIGNUPS_VERIFY:** This setting determines whether or not new accounts must verify their email address before being able to login to Vaultwarden. Possible values: true / false.
- **SIGNUPS_VERIFY_RESEND_TIME:** If SIGNUPS_VERIFY is set to true, this value specifies how many seconds a user must wait before another verification email can be sent.
- **SIGNUPS_VERIFY_RESEND_LIMIT:** If SIGNUPS_VERIFY is set to true, this value specifies the maximum number of times an email verification may be re-sent.
- **SIGNUPS_DOMAINS_WHITELIST:** This setting is a comma separated list of domains that can register for Vaultwarden accounts, even if SIGNUPS_ALLOWED is set to false. This is useful for when your Vaultwarden accounts are to be used specifically by email addresses whose domains you control.


### **SMTP Settings**
- **SMTP_HOST:** This is your SMTP mailserver.
- **SMTP_FROM:** This is the email address messages will be sent from.
- **SMTP_FROM_NAME:** The name you wish to appear as the email account name on sent messages
- **SMTP_SECURITY:** The security method used by your SMTP server. Possible values: “starttls” / “force_tls” / “off”.
- **SMTP_PORT:** This is the SMTP port used by your mail server. Possible values: 587 / 465.
- **SMTP_USERNAME:** This is the login for your SMTP mail server.
- **SMTP_PASSWORD:** This is the password for your SMTP credentials.
- **SMTP_AUTH_MECHANISM:** This is the SMTP authentication mechanism of your mail server. Possible values: “Plain” / “Login” / “Xoauth2”.

I'm using **Vaultwarden** which is an opensource project. It is not owned by Bitwarden. They're an unofficial bitwarden compatible server written in Rust.

Save your `docker-compose.yml` file and exit back to your bitwarden directory.

## Verify and launch the container 

to verify the docker compose files has no issues use `docker compose config` Once you have the `docker-compose.yml` file you can run `docker-compose up -d` which will create the volume, download the image and start the container in the background 

To verify the status of all running containers, run the docker-compose command:

```bash
docker-compose ps (OR) docker ps -a
```

If something goes wrong, you can check the logs for each container using the docker logs command and specify the service name. For example, to check the log of the Nginx Proxy Manager container, run the following command:

```bash
docker logs $container (OR) docker compose logs -f
```

## Accessing the bitwarden dashboard

If all is well, you can locally view your Bitwarden Server by navigating to http://localhost:PORT. Or from another machine by using your ip address instead of localhost

You should see something that looks like the following.

![web](/assets/docker/vault/Web.png){: width="800" height="500" }

Finally, you'll just need to register for an account on your new hosted instance.

Click the Create Account button

Then fill out your details. If you have an existing Bitwarden account, you'll still have to create a new account on this instance. You can then Export and Import your credentials.

![export](/assets/docker/vault/export.png){: width="800" height="500" } || ![import](/assets/docker/vault/import.png){: width="800" height="500" }

Browsers like chrome, firefox ... also has the inbuilt vaults to store the credentials you can also export the credentials in a csv/json format and export 

You can use the [Nginx proxy manager]({% post_url 2023-01-19-npm %}){:target="_blank"} to create the reverse proxy and configure the lets-encrypt SSL certificate to remove the SSL warnings

you can use the bitwarden clients applications/browser extensions to access the stored credentials [Application](https://bitwarden.com/download/){:target="_blank"} [firefox extension](https://addons.mozilla.org/en-US/firefox/addon/bitwarden-password-manager/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search){:target="_blank"} [chrome extension](https://chromewebstore.google.com/detail/bitwarden-free-password-m/nngceckbapebfimnlniiiahkandclblb){:target="_blank"}

> For More details please check the [github issues](https://github.com/dani-garcia/vaultwarden/discussions){:target="_blank"} [discussions](https://vaultwarden.discourse.group/){:target="_blank"}
{: .prompt-info}