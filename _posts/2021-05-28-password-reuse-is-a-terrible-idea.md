---
title: 'Password Reuse is a Terrible Idea'
date: '2021-05-28T21:35:27-04:00'
layout: post
author: LukeTech
permalink: /2021/05/password-reuse-is-a-terrible-idea/
tags: [data breach, password, Member Article]
---

![](/assets/uploads/2021/05/password.png)
> "76 percent of employees at the world’s largest companies reuse passwords across personal and professional accounts"
> 
> <cite>Chip Witt, SpyCloud</cite>

I found this on [Help Net Security](https://www.helpnetsecurity.com/2021/02/15/password-reuse-risk/) and it is a statistic I've been seeking for some time. Let me tell you why that is a problem from several angles.

Password reuse is the tendency for people to use the same or similar passwords across different accounts, and between personal and work life. There is a significant risk to individuals and companies because there are hundreds of millions of breached credentials floating around from historic and, eventually, future data breaches. This overlap can be massive and a breach on one side can affect the other.

Data breaches may contain lists of usernames and passwords. They are collected over the years, de-duplicated, sorted, and packaged into "combo lists." These are huge, multi-gigabyte archives containing hundreds of millions of searchable credentials.

One way attackers target companies is by creating lists of high-value individuals and search the combo lists for matches. If they find an individual's personal email password, there is a statistically high likelihood the victim uses the same or derivative password in their work email – or vice-versa. See the problem with this?

From a security defense perspective, it can be beneficial to acquire these combo lists and search for employee credentials by domain name, for example, “@company.com,” or for obvious first name/last combinations. Then, compare the results to the employee's work accounts. Some employees use the same password years after it was involved in a data breach, and there is no happy ending to that story.

---

Another scenario involves the company security team legitimately auditing employee's credentials. These are stored in a Windows Active Directory domain controller file (NTDS.dit), the most common directory platform used in medium and large organizations. Usernames and passwords are extracted from this database. In most cases, they are encrypted (good), so it takes effort to “crack” the encrypted hash of the passwords into their plaintext. I’ll avoid the nitty-gritty details here, but it involves significant computing resources and is essentially limited by time and money.

Fast forward, and now the security team has a potentially large number of plaintext passwords. They search the combo lists and inevitably match some personal accounts having the same or similar password as some employees.

Notice there are several ethical and legal issues. First, a legal gray area of possessing stolen data from data breaches. The usernames and passwords may be construed as hacking tools as some will be valid. Finally, an ethical issue exists in possessing a large number of credentials to employee’s personal accounts.

While I support auditing an organization’s passwords this way, it is fraught with risk. If mishandled or compromised, the loss of domain credentials (cracked from the NTDS.dit database) could lead to an inadvertent data breach of the company and its employees. It very well contains passwords not found in other breaches and contains the keys to many kingdoms – both personal and professional accounts.

---

The main takeaway is to avoid using the same or similar passwords across any two or more accounts, and across personal and professional roles. A password manager really helps here. Multi-factor authentication adds a strong layer of protection.

Since there is overlapping password reuse, security teams must practice a higher level of diligence and operational security (OPSEC) than almost anything else they do when auditing passwords in Active Directory. Every step of the way from extracting the NTDS.dit, moving it to an air-gapped system via encrypted storage, cracking the hashes, storing the plaintext passwords, comparing to combo lists, handling the results, and sanitizing the system and storage media when finished.