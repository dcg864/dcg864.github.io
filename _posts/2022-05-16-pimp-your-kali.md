---
title: 'Pimp Your Kali'
date: '2022-05-16T19:48:58-04:00'
layout: post
author: jazzy_j
permalink: /2022/05/pimp-your-kali/
image: /assets/uploads/2022/05/Kali-Linux-2020.4-is-ready-for-download-1.jpg
tags: [customizing kali, tools, Member Article]
---

There are many people who download, install, and run [Kali](https://www.Kali.org/) in it's default state. Out of the box, Kali comes with a plethora of great tools that we are all familiar with. However, there's not a lot of options for themes, so I'd like to share my setup and some of the tools I use. Whether it be for CTFs, your home lab, a particular certification course, or work, these tools are great to have in your arsenal.

## Terminator

Terminator is a great emulator that has a ton of functionalities like being able to split horizontally, split vertically, layout savings, and creating a custom title to the terminal.

To install, open up a terminal and run:

```
sudo apt install terminator
```

As of this writing, Kali has now changed their default shell to zsh. You don't have to use zsh if you don't want to, but if you do, you can switch your shell by doing the following:

```
chsh -s $(which zsh)
```

Now, if you're like me, you may want or need something that's more aesthetically pleasing to the eye. For this, I use EliverLara's terminator-themes that can be found[ ](https://github.com/EliverLara/terminator-themes)on Github (<https://github.com/EliverLara/terminator-themes>).

## Candy icon theme

![](/assets/uploads/2022/05/image.png)
I actually just came across this gem fairly recently and I absolutely love it! Created by EliverLara, Candy is a gradient style theme that is colorful and very different from most of the icon themes available. The repo can be found here <https://github.com/EliverLara/candy-icons>. To install, simply download the .zip file from the Github page into your Downloads directory.

Now, extract the contents:

```
sudo tar xvf ~/Downloads/candy-icons.tar.xz
```

**CD** into the Downloads directory

```
cd ~/Downloads
```

You will now need to run the following as root, so the easiest way is to do the following:

```
sudo -s
```

Now, place the **candy-icons** directory into **/usr/share/icons**:

```
mv candy-icons /usr/share/icons
```

Candy is now installed on the computer, but it is not in use yet. You must change the default system icons by going to **System Settings -&gt; Appearance**, click on the **Icons** tab and select candy-icons:

![](/assets/uploads/2022/05/image-3.png)
# Tools

What I love about being on this side of Security is that there are tons of tools at our disposal. This could seem overwhelming at first and can lead to you asking yourself "Am I choosing the right tool?", "Do I need to use all of them?", etc. My advice is to start with a small amount and get familiar with them and over time you may decide you want to add more tools to your bag of tricks. Then again, you may be perfectly content with the ones you're using. There's really no wrong or right answer!

This won't be an exhaustive list but here are some worth mentioning:

### Threader3000 - Multi-threaded Port Scanner

```
pip3 install threader3000
```

### fcrackzip - Password-protected ZIP file cracker

```
sudo apt install fcrackzip
```

### hash-identifier - Identifies different types of hashes

```
sudo apt install hash-identifier
```

### Name-That-Hash - Similar to hash-identifier but provides better results

```
sudo apt install name-that-hash
```

### pspy - Tool that enumerates processes without the need for root permissions

<https://github.com/DominicBreuker/pspy>

### Evil-WinRM - Windows Remote Management shell

<https://github.com/Hackplayers/evil-winrm>

### Kerbrute - Tool to perform Kerberos pre-auth bruteforcing

```
pip3 install kerbrute
```

### PEASS - Privilege Escalation Awesome Scripts Suite

**linpeas:** <https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS>

**winpeas:** <https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS>

### Dufflebag - Search for exposed EBS volumes for secrets

```
sudo apt install dufflebag
```

### S3Scanner - Scan for open S3 buckets and dump the secrets

```
sudo apt install s3scanner
```

Again, this isn't an exhaustive list but they are great tools to start out with. Hopefully this will benefit you as it has benefited me.

Happy Hacking!