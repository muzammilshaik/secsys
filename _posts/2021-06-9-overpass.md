---
author: [muju]
title: "overpass"
description: ""
categories: ["CTF", "THM"]
tags: ["linpeas"]
image: /assets/ctf/overpass/overpass_2.png
toc: false
---

we are going to solve an online challenge called the overpass. As we find the two open ports one is being ssh on port 22 and the other HTTP on port 80. we find the private ssh key and we will crack the password with john tool through which we will ssh into the box and we will do some basic enumeration in the box to get the root user with all being said lets get started

First we will start with the nmap to find the open ports 

`nmap -sC -sV -oN nmap/initial_scan $IP`

![overpass1](/assets/ctf/overpass/overpass_1.png){: width="800" height="500" }

Let’s take a look at the port 80. we have Overpass page saying that a secure password manager with support for windows, Linux, macos and more

![overpass2](/assets/ctf/overpass/overpass_2.png){: width="800" height="500" }

Let’s fire up the gobuster to find the directories and the pages present in it and save the output yo gobuster.log

`gobuster dir -u http://10.10.44.66/ -w /usr/share/wordlist/dirbuster/directorylist-2.3-medium.txt -o gobuster.log `

![overpass5](/assets/ctf/overpass/overpass_5.png){: width="800" height="500" }

Lets open the /admin page we find a simple login page nothing interesting there. let’s open the page source we will find the login.js java script there open the java script file as we send the request sessiontoken = something.  we will get to /admin

![overpass6](/assets/ctf/overpass/overpass_6.png){: width="800" height="500" }

Let send the request through the curl by adding the cookie sessiontoken=anything

`curl "http://10.10.44.66/admin/" --cookie "SessionToken=anything"`

![overpass7](/assets/ctf/overpass/overpass_7.png){: width="800" height="500" }

Here we got an private ssh key. copy the private ssh key to a file and change the permission of the private key and lets try to ssh it with the key 

![overpass8](/assets/ctf/overpass/overpass_8.png){: width="800" height="500" }

As the private key is asking for password lets try to crack it with the john too. For this john has a ssh2john,py script first we have to execute a script on private key and save it to a file 

`ssh2john.py id_rsa > forjohn.txt`

![overpass9](/assets/ctf/overpass/overpass_9.png){: width="800" height="500" }

Now run the john tool we will get the password as james13

![overpass10](/assets/ctf/overpass/overpass_10.png){: width="800" height="500" }

As we have the private ssh key and the password lets ssh into it

![overpass11](/assets/ctf/overpass/overpass_11.png){: width="800" height="500" }

we are now in as a low level user let’s do some enumeration to get the high level user. Let’s start the HTTP server and upload the linpeas.sh and run it

`python3 -m http.server && chmod +x linpeas.sh && ./linpeas.sh`

![overpass12](/assets/ctf/overpass/overpass_12.png){: width="800" height="500" }

A little bit of enumeration our user can run curl as root only on overpass.thm domain

![overpass13](/assets/ctf/overpass/overpass_13.png){: width="800" height="500" }

Let’s modify the domain address to our ip address so that we can get the root user

![overpass14](/assets/ctf/overpass/overpass_14.png){: width="800" height="500" }

make the request is going through the downloads/src/buildscript.sh let’s make the directories for it and make the bildscript.sh to run in bash and start the http server

`mkdir -p downloads/src && 
cd $_ && 
vi buildscript.sh &&
sudo python3 -m http.server`

![overpass15](/assets/ctf/overpass/overpass_15.png){: width="800" height="500" } | ![overpass-script](/assets/ctf/overpass/overpass_script.png){: width="800" height="500" }

These script will  gives us the accessible root privileges through bash binary setuid

![overpass16](/assets/ctf/overpass/overpass_16.png){: width="800" height="500" }

That's all for to day hope you like the box see you i next tutorial until then good bye have  a great day
