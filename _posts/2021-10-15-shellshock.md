---
author: [muju]
title: "Shell Shock"
description: ""
categories: ["CTF", "THM"]
tags: ["john", "metasploit"]
image: /assets/ctf/0day/0day_logo.jpeg
---

## what is shell shock vulnerability

Shellshock is a security bug causing Bash to execute commands from environment variables unintentionally. In other words if exploited the vulnerability allows the attacker to remotely issue commands on the server, also known as remote code execution. Even though Bash is not an internet-facing service, many internet and network services such as web servers use environment variables to communicate with the server’s operating system.Since the environment variables are not sanitized properly by Bash before being executed, the attacker can send commands to the server through HTTP requests and get them executed by the web server operating system.

## Scanning

Let’s scan with the nmap to find the open services on the box

`nmap -sC default scripts -sV enumerate version -oN output out format ip`

![0day1](/assets/ctf/0day/0day_1.png){: width="800" height="500" }

We have two services open one is being SSH and the other is HTTP. As the web server is open let’s fire up the gobuster to find the directories and the nikto to find vulnerabilities and mis-configurations on the server.

`gobuster dir -u url -w wordlist -x extensions -o output` `nikto -h url`

![0day2](/assets/ctf/0day/0day_2.png){: width="800" height="500" } || ![0day3](/assets/ctf/0day/0day_3.png){: width="800" height="500" }

looking at the gobuster results it seems to be /backup is interesting. Let’s check what is present in the backup directory

![0day5](/assets/ctf/0day/0day_5.png){: width="800" height="500" }

## John The Ripper

John the Ripper is a fast password cracker Its primary purpose is to detect weak Unix passwords we can crack the private key with the john for this we need to run the script to change the private key for john compatibility

`ssh2john.py ssh/privatekey > output.txt`

![0day6](/assets/ctf/0day/0day_6.png){: width="800" height="500" }

Run the john against the private key and we have a password of an private key. let’s login with the private key but we couldn’t login because we didn’t have the user password

![0day7](/assets/ctf/0day/0day_7.png){: width="800" height="500" }

Let’s see what we can do with it, looking at the nikto results /cgi-bin/test.cgi seems to be vulnerability with the shell shock vulnerability. It’s time to start the metasploit framework

## Metasploit Framework

We are using the metasploit framework use the exploit 

```
msf > use exploit/multi/http/apache_mod_cgi_bash_env_exec
msf > exploit(apache_mod_cgi_bash_env_exec) > set TARGET < target-id >
msf > exploit(apache_mod_cgi_bash_env_exec) > show options
    ...show and set options...
msf exploit(apache_mod_cgi_bash_env_exec) > run

0day_8
```

![0day8](/assets/ctf/0day/0day_8.png){: width="800" height="500" }

Ones we run it we have a meterpreter shell opened as a low level user


## Privilege Escalation

Lets upload the linpeas to escalate the privileges

![0day9](/assets/ctf/0day/0day_9.png){: width="800" height="500" }

by looking at the results we can say that the Linux version is too old we can exploit it using the CVE-2015-1328

## CVE-2015-1328

Description The overlayfs implementation in the Linux (aka Linux kernel) package before 3.19.0-21.21 in Ubuntu through 15.04 does not properly check permissions for file creation in the upper filesystem directory, which allows local users to obtain root access by leveraging a configuration in which overlayfs is permitted in an arbitrary mount namespace.

upload the exploit and run it

![0day10](/assets/ctf/0day/oday_10.png){: width="800" height="500" }

We are root now ! HOPE YOU LIKE THE BOX HAVE A GOOD DAY

