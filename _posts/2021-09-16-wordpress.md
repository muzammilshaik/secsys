---
author: [muju]
title: "Wordpress"
description: ""
categories: ["CTF", "THM"]
tags: ["streghide", "wpscan", "metasploit", "find"]
image: /assets/ctf/wordpress/wordpress-logo.png
---

## scanning

Let’s scan the host to find the open ports 

![wordpress1](/assets/ctf/wordpress/wordpress_1.png){: width="800" height="500" }

We have three services open one is being an SSH HTTP & SMB  let’s check the web server

![wordpress4](/assets/ctf/wordpress/wordpress_4.png){: width="800" height="500" }

nothing much interesting hear. we have a billy as the user let’s check the samba share with this user name

![wordpress2](/assets/ctf/wordpress/wordpress_2.png){: width="800" height="500" }

As the share contains only three two jpg files and one mp4. Download it locally and check whether the jpg files contains any data inside it with streghide

## [streghide](http://steghide.sourceforge.net/download.php)

[Steghide](http://steghide.sourceforge.net/download.php) is a steganography program that is able to hide data in various kinds of image- and audio-files. The color- respectivly sample-frequencies are not changed thus making the embedding resistant against first-order statistical tests.

Steganography literally means covered writing. Its goal is to hide the fact that communication is taking place. This is often achieved by using a (rather large) cover file and embedding the (rather short) secret message into this file. The result is a innocuous looking file (the stego file) that contains the secret message.

![wordpress3](/assets/ctf/wordpress/wordpress_3.png){: width="800" height="500" }


## wpscan

WPScan is an open source WordPress security scanner. You can use it to scan your WordPress website for known vulnerabilities within the WordPress core, as well as popular WordPress plugins and themes.

Since it is a WordPress black box scanner, it mimics a real attacker. This means it does not rely on any sort of access to your WordPress dashboard or source code to conduct the tests. In other words, if WPScan can find a vulnerability in your WordPress website, so can an attacker.

WPScan uses the vulnerability database called wpvulndb.com to check the target for known vulnerabilities. The team which develops WPScan maintains this database. It has an ever-growing list of WordPress core, plugins and themes vulnerabilities.

![wordpress5](/assets/ctf/wordpress/wordpress_5.png){: width="800" height="500" }
![wordpress6](/assets/ctf/wordpress/wordpress_6.png){: width="800" height="500" }

With the help of the wpscan we can got the users names and we dont have a valid password to login let’s bruteforce to find the valid password

## Brute Force

![wordpress7](/assets/ctf/wordpress/wordpress_7.png){: width="800" height="500" }

We have a valid username and password login with that creds into the wordpress

![wordpress8](/assets/ctf/wordpress/wordpress_8.png){: width="800" height="500" }

## WordPress Crop-image Shell Upload

As the above WordPress version is 5.0 we have WordPress Crop-image Shell Upload vulnerability present in this version. This exploits a path traversal and a local file inclusion vulnerability on WordPress versions `5.0.0 and <= 4.9.8`.

The crop-image function allows a user, with at least author privileges, to resize an image and perform a path traversal by changing the `_wp_attached_file` reference during the upload.

The second part of the exploit will include this image in the current theme by changing the `_wp_page_template` attribute when creating a post. This exploit module only works for Unix-based systems currently.

## metasploit

Let’s fire up the metasploit framework and run the crop image exploit by setting the WordPress login credentials. if you don't have metasploit framework you can install it by using this single command in your terminal

```
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && chmod 755 msfinstall && ./msfinstall
```

![wordpress9](/assets/ctf/wordpress/wordpress_9.png){: width="800" height="500" }

one’s you set all the requirements run it and we have a meterpreter shell opened as a low level user

## Enumeration

with the help of find command lets check the enumerate of all binaries having SUID permission. `find / -perm -u=s -type f 2>/dev/null`

![wordpress10](/assets/ctf/wordpress/wordpress_10.png){: width="800" height="500" }

The /usr/sbin/checker file seems to be interesting and all the  other files are quite common. let’s trace the checker with ltrace and we export admin = 1 and we have a nice root shell hear

![wordpress11](/assets/ctf/wordpress/wordpress_11.png){: width="800" height="500" }

That’s all for to day hope you like the box. HAVE A GREAT DAY