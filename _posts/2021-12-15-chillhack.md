---
author: [muju]
title: "Chill Hack"
description: ""
categories: ["CTF", "THM"]
tags: ["Pwncat", "zip", "john"]
image: /assets/ctf/chill_hack/chill_hack.png
---

## Reconnaissance

Let’s do  a quick reconnaissance to find the information about the box by doing a nmap scan to find the open ports

![chill1](/assets/ctf/chill_hack/chill_1.png){: width="800" height="500" }

As we have ftp port open by seeing at the namp results we have anonymous login allowed lets login as anonymous

![chill2](/assets/ctf/chill_hack/chill_2.png){: width="800" height="500" }

We have a successful login and we have a note.txt hear download it with get it and cat it out as we have usernames hear and nothing much. Let’s have a look at the port 80

![chill3](/assets/ctf/chill_hack/chill_3.png){: width="800" height="500" }

we have a static web page. run the gobuster and nikto to find the directories present in it

![chill4](/assets/ctf/chill_hack/chill_4.png){: width="800" height="500" } || ![chill5](/assets/ctf/chill_hack/chill_5.png){: width="800" height="500" }

By looking at the gobuster and nikto results /secrets seems to be interesting open the /secrets page we have commands execution hear run the ls to check the files present in it

![chill6](/assets/ctf/chill_hack/chill_6.png){: width="800" height="500" } || ![chill7](/assets/ctf/chill_hack/chill_7.png){: width="800" height="500" }

we have an error when we executed the ls command we have to bypass the filter to get the command execution. send the request to the brup and so that we can easily modify the request

## filter’s bypass

Lets fire up the burp suite and catch the request so that we can play around it 

![chill8](/assets/ctf/chill_hack/chill_8.png){: width="800" height="500" }

we have a 200 response that means we have successfully bypassed the filters lets cat the index file to see the filters present in it so that we can get the reverse shell

![chill9](/assets/ctf/chill_hack/chill_9.png){: width="800" height="500" }

the filter seems to block the tools like python bash php perl rm cat head tail python3 more less sh and ls commands that's why when we executed the ls command we get an error. Since we cannot execute the shell scripts we can by pass this filters by creating an exploit in our local meacine start the simple http server and curl the exploit and send it to the bash to get the reverse shell. bash is also get blocked due to filter we can bypass it by send bash and the it wont get stopped by filer

## Getting an shell

lets create the exploit to get the reverse shell connection 

![chill11](/assets/ctf/chill_hack/chill_11.png){: width="800" height="500" } || ![chill0](/assets/ctf/chill_hack/chill_10.png){: width="800" height="500" }

stat the listener service when the code get executed it should give us a no response seems to be interesting lets look at the listener service we have a reverse shell as www-data

![chill12](/assets/ctf/chill_hack/chill_12.png){: width="800" height="500" }

lets check what permission we have as a www-data user and we can run the helpline script and we logged in as another  user we can conform it by seeing at the id output

![chill13](/assets/ctf/chill_hack/chill_13.png){: width="800" height="500" }

## SSH Keys

lets create a ssh key and upload it in the user authorized_keys so that we can ssh into the box

We have a successful login to ssh and by doing some basic enumeration we came to know that another port is listen 9001. forward the port locally and locate it at https://localhost:9001

![chill14](/assets/ctf/chill_hack/chill_14.png){: width="800" height="500" }

## bypassing login

![chill16](/assets/ctf/chill_hack/chill_16.png){: width="800" height="500" }

We don’t have the credentials to login. lets save the request to a file and run the sqlmap to find the bypass payload to login we have bypassed it with this payload and we have a login

![chill17](/assets/ctf/chill_hack/chill_17.png){: width="800" height="500" } || ![chill18](/assets/ctf/chill_hack/chill_18.png){: width="800" height="500" }

## Steganography

we have only some text on the page look in the dark! you will find your answer.  and we have a image lets wget it to download it and use steghide to see any information is hidden in it

![chill19](/assets/ctf/chill_hack/chill_19.png){: width="800" height="500" }

## John

we have find that some information is hidden in it we need to have the credentials to extract it from the jpg file am using joh tool to extract the information from it. first we have to use zip to john tool against the backup.zip file ehich was extracted from the jgp file and save the output hash to a file and run the john tool against the hash to get the passpharse to extract the information from the backup.zip

![chill20](/assets/ctf/chill_hack/chill_20.png){: width="800" height="500" }

## Privilege escalation

run the id command by looking at it we came to know that we are in a DOCKER container and If the binary is allowed to run as superuser by `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.

![chill21](/assets/ctf/chill_hack/chill_21.png){: width="800" height="500" }

That’s it for today hope you like the box subscribe for more updated content fell free to drop a comment  and HAVE A GREAT DAY !
