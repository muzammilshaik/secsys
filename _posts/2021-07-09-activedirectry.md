---
author: [muju]
title: "Active Directory"
description: ""
categories: ["CTF", "THM"]
tags: ["impacket", "kerbrute", "psexec"]
image: /assets/ctf/ad/dc_logo.png
toc: false
---

Most of the Corporate networks run off of AD. We are going to exploit a DOMAIN CONTROLLER. With the kerbrute and impacket tools to enumerate the box .  Let’s scan for the open ports with the nmap

![dc1](/assets/ctf/ad/dc_1.png){: width="800" height="500" } || ![dc0](/assets/ctf/ad/dc_0.png){: width="800" height="500" }

## Impacket

Installing Impacket:

First, you need to clone the repo with: `git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket` This will clone Impacket to `/opt/impacket/` once the repo is cloned, you will notice several install related files, requirements.txt, and setup.py. Setup.py is commonly skipped during the installation. It’s key that you DO NOT miss it. So let’s install the requirements: `pip3 install -r /opt/impacket/requirements.txt`

Once all the python modules are installed, we can then run the python setup install script `cd /opt/impacket/ && python3 ./setup.py install`

After that, Impacket should be correctly installed and then download the kerbrute `https://github.com/ropnop/kerbrute`

![dc2](/assets/ctf/ad/dc_2.png){: width="800" height="500" }

we have got the users account and the svc-admin user names of the spookysec.local. Let’s run a tool from IMPACKET to collect the hash of a svc-admin account

![dc3](/assets/ctf/ad/dc_3.png){: width="800" height="500" }

we got the hash save it to a file and crack the hash with the hashcat

![dc4](/assets/ctf/ad/dc_4.png){: width="800" height="500" } || ![dc4](/assets/ctf/ad/dc_4_hashcat.png){: width="800" height="500" }

We have a valid user name and the password let’s check the shares with the SMBCLIENT

![dc5](/assets/ctf/ad/dc_5.png){: width="800" height="500" }

As the backup account password is stored in base64 format  let’s decrypt it by `echo " hash " | base64 -d` 

![dc6](/assets/ctf/ad/dc_6.png){: width="800" height="500" }

As we have the backup account le’s try to dump the HASHES by dumping the NTFS.DIT by using the impacket tool

![dc7](/assets/ctf/ad/dc_7.png){: width="800" height="500" }

We got the Administrator hash. We are going to PASS THE HASH attack to authenticate with out the password as a Administrator 

![dc8](/assets/ctf/ad/dc_8.png){: width="800" height="500" }

That’s all for to day hope you like the box. HAVE A GOOD DAY
