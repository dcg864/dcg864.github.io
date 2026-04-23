---
title: AutoRpt
date: '2021-06-27T11:25:43-04:00'
layout: post
author: overcast
permalink: /2021/06/reporting-tools-july-meeting-topic/
image: /assets/uploads/2021/06/autorpt-sm.png
tags: [automation, pentest, report writing, Meeting Announcement, Meeting Recap]
---

## Meeting Recap

I presented a tool called [AutoRpt ](https://github.com/BenAcord/AutoRpt)to automate the exam setup and report writing process. This was initially coded during my preparation for the OSCP exam and I should probably own the fact it was a shiny object distraction.

For a bit of context, the OSCP exam is split into two timebound sessions. The first 24-hours is the penetration test and the second 24-hours is for report writing and submission. During the OSCP exam the test taker has several options for reporting workflow. Offensive Security provides [suggested document templates for Word and Writer.](https://help.offensive-security.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide) These can be edited during the exam window or after the VPN drops in the second 24-hour period. Personally, I've never done well notetaking on the fly in Writer or Word. When I've tried this in the past the result is a crazy mess that takes significant time to clean up. Editing the report in the suggested templates during the exam is somewhat of an efficiency gain but comes with its own challenges such as: formatting text, code blocks, back tracking notes for a rabbit trail, time commitment, etc. all of which impede the act of performing the pentest methodology during the exam window. Remember the clock doesn't slow if you fall into a writing rabbit hole.

A few things happened that caused me step back and reevaluate the report writing methodology and ultimately code AutoRpt. The first was really liking AutoRecon as my initial recon tool of choice with one caveat, I didn't like how it put a report subdirectory under each results target IP address. The second was switching from notetaking in Writer/Word to plaintext, which didn't last long, and I soon pivoted to markdown using [Obsidian ](https://obsidian.md/)(a amazing tool...and you're welcome). The third and final motivation happened when I came across noraj's GitHub repo [OSCP-Exam-Report-Template-Markdown](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown) to generate the PDF and 7z file for submission to OffSec. That's when it all clicked and I started coding.

AutoRpt has two parameters. The first, called **startup**, creates an exam working directory and stubbed report files for either an exam (eg. OSCP) or for one-off training (eg. the OffSec lab, Hack the Box, Try Hack Me, PortSwigger's Web Security Academy, etc.).

Continuing with the OSCP example, run `autorpt startup` and pick option 1.

![](/assets/uploads/2021/07/2021-07-exam-startup.png)
This creates a subdirectory in your current working directory of the exam name and a subdirectory for report. The markdown files in the report directory have specific naming standards with a prefix of zero through six. This is the order each document will merge into a single markdown when producing the PDF. 0- is the executive summary and 6-closing contains any appendices. All of the markdown files have helpful text stubbed in each section to guide your thought process. This makes it easy to remove a system section markdown file if it wasn't completed during the exam.

When the exam starts, navigating into the exam directory and running AutoRecon will create its standard directory structure and output files from the various tools it runs. The example in the following image is of Metasploitable2 so no exam spoilers here.

![](/assets/uploads/2021/07/2021-07-exam-during.png)
Throughout the engagement update the \[1-5\]\*.md files as you progress through the systems. Since markdown is text you can edit the files any way that suites you; vim, emacs, gedit, echo, Obsidian. I won't judge. Once the pwnage is complete edit the 0-execsummary and 6-closing files accordingly.

Now comes the need for the second parameter of AutoRpt, **finalize**, which produces the PDF and 7z similar to that of the noraj project. Verify the PDF content is correct. If a typo or change is discovered modify the original markdown file as needed and rerun `autorpt finalize`.

![](/assets/uploads/2021/07/2021-07-exam-finalize-1.png)
There is a lot more I want to do with this project to mature it going forward. And several people at the meeting suggested functionality enhancements which are now GitHub issues. If you have an idea or find a bug please submit a new issue to the project.