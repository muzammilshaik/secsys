---
author: [muju]
title: "Brute Force"
description: ""
categories: ["CTF", "THM"]
tags: ["hydra", "john"]
image: /assets/ctf/brute/logo.png
---

## Reconnaissance

let’s do the quick scanning to find the open ports on the box

![brute1](/assets/ctf/brute/brute_1.png){: width="800" height="500" }

nmap -sc -sV -oN file_name ip

nmap -SC for default cripts aand -sV for enumerate version -oN for simple nmap format and the ip address you want to scan 
there are two ports open one is being SSH and the other is web server HTTP, ssh is running on version openssh 7.6 and the apache 2.4.29. As the web server is open scan for the directories present in it with the gobuster

![brute2](/assets/ctf/brute/brute_2.png){: width="800" height="500" }

gobuster dir -u url -w wordlist -o output

gobuster dir for specifying the gobuster in directory mode and -u for the URL you want to scan and the -o for output the scanning results
we have got the results /admin login  page lets check the page

![brute3](/assets/ctf/brute/brute_3.png){: width="800" height="500" }

## Hydra

hydra – Very fast network logon crackerHydra is a parallelized login cracker which supports numerous protocols to attack. It is very fast and flexible, and new modules are easy to add. This tool makes it possible for researchers and security consultants to show how easy it would be to gain unauthorized access to a system remotely.

Hydra is a tool to guess/crack valid login/password pairs – usage only allowed for legal purposes.

we are going to brute force the admin panel to gain the access into the box

![brute4](/assets/ctf/brute/brute_4.png){: width="800" height="500" }

we found the admin credentials by brute forcing the admin panel with the rockyou wordlist, login with the credentials into the admin panel

![brute_5](/assets/ctf/brute/brute_5.png){: width="800" height="500" }

## john

first  run the ssh2john python script for the rsa private key and save the output format to a file. then run the run the john on the output file to start the john you can use custom wordlist if you want or the you can use the custom wordlist which came along with the john the ripple 

![brute6](/assets/ctf/brute/brute_6.png){: width="800" height="500" }

we got the password of the rsa private key and we have the ssh port open lets login into the SSH. we got logged in into the box and we have successfully logged in

## Enumeration

if we check the permission of the user we can run the cat as sudo, If the binary is allowed to run as superuser by `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access. `sudo cat /etc/shadow`

![brute7](/assets/ctf/brute/brute_7.png){: width="800" height="500" }

we got the hash of the root user, let’s crack it with the john. first save the hash to a file and sun the john tool in this case am not using any wordlist just providing the ash to the john to crack it

![brute8](/assets/ctf/brute/brute_8.png){: width="800" height="500" }

we got the root credentials

that’s all for today hope you like the box, have a GREAT DAY.

