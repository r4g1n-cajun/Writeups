### SolidState: 1

#### About
  - Name: SolidState: 1
  - Date release: 12 Sep 2018

  - Author: [Ch33z_plz](https://www.vulnhub.com/author/ch33z_plz,242/)
  - Series: [SolidState](https://www.vulnhub.com/series/solidstate,180/)

#### Description
It was originally created for HackTheBox

---------------

# Walkthrough

## Enumeration

***Nmap***

The first thing that stands out is the the JAMES services running (we've seen these before):

```
root@kali:~# nmap -Pn -sC -sV 192.168.165.136
Starting Nmap 7.70 ( https://nmap.org ) at 2018-11-25 17:20 EST
Nmap scan report for 192.168.165.136
Host is up (0.00073s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey:
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp  open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (192.168.165.135 [192.168.165.135]), PIPELINING, ENHANCEDSTATUSCODES,
80/tcp  open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp open  pop3    JAMES pop3d 2.3.2
119/tcp open  nntp    JAMES nntpd (posting ok)
MAC Address: 00:0C:29:75:23:E9 (VMware)
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.52 seconds
```

So, naturally we want to check if the admin port is accessible and.... of coarse it is:

```
rroot@kali:~# nmap -Pn -sC -sV -p 4555 192.168.165.136
Starting Nmap 7.70 ( https://nmap.org ) at 2018-11-25 17:20 EST
Nmap scan report for 192.168.165.136
Host is up (0.00045s latency).

PORT     STATE SERVICE     VERSION
4555/tcp open  james-admin JAMES Remote Admin 2.3.2
MAC Address: 00:0C:29:75:23:E9 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.46 seconds
```

***Netcat***

The next step is to connect to the admin port with netcat and check if it's still using the default credentials (root:root), which it is:

```
root@kali:~# nc 192.168.165.136 4555
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
```
