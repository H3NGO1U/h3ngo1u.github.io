---
layout: post
title:  "Try Hack Me - Industrial Intrusion Competition Write-Up"
---
# THM Industrial Intrusion WriteUp

Engagement challenge

*This engagement aims to find a way to open the gate by bypassing the badge authentication system.
The control infrastructure may hold a weakness: Dig in, explore, and see if you have what it takes to exploit it.
Be sure to check all the open ports, you never know which one might be your way in!*

Scanning all ports can take time, 

We can use nmap to scan all open ports, I use this shell script:

```bash
!/bin/bash
if [ -z "$2" ]; then
echo "Usage: scan <IP> <target_file>"
exit 1
fi
IP=$1
target_file=$2
ports=$(nmap -p- --min-rate=1000 -T4 $IP | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
if [ -z "$ports" ]; then
echo "No open ports found on $IP"
exit 1
fi
echo "Scanning open ports: $ports"
nmap -sC -sV -p$ports $IP -oN $target_file
echo "Finish scanning and saved result to file"
```

Which first scans all ports and then, only for the open ports it uses -sC and -sV (script scan and version detection).

So open ports are 22,80,102,1880,8080,44818.

22/tcp    open  ssh           OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)

80/tcp    open  http          Werkzeug/3.1.3 Python/3.12.3

102/tcp   open  iso-tsap      Siemens S7 PLC

8080/tcp  open  http-proxy    Werkzeug/2.3.7 Python/3.12.3

44818/tcp open  EtherNetIP-2

In the 80 port we see this interface:

![lo](/assets/img/imgs-2025-06-27-thm-industrial-instrusion-writeup/image.png)

In 8080 port we have this login system:

![8080](/assets/img/imgs-2025-06-27-thm-industrial-instrusion-writeup/image1.png)

Tried the default creds openplc, openplc but that didn’t work.

The description implies that we should be looking for the unstandart ports, 

![1880](/assets/img/imgs-2025-06-27-thm-industrial-instrusion-writeup/image%202.png)

In the ui endpoint we can see:

![ui](/assets/img/imgs-2025-06-27-thm-industrial-instrusion-writeup/image%203.png)

By turning them off we bypass the gate and get the flag 🎊

![Flagy](/assets/img/imgs-2025-06-27-thm-industrial-instrusion-writeup/image%204.png)
