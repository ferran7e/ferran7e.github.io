---
layout: post
title: Ali Hadi's Challenge #3 | Solved!
categories: [Writeup, Forensics]
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/-QZe9p5zbLs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Today I'll be discussing and walking through my solution to Ali Hadi's "Mystery Hacked System" challenge located on his website, [here](https://www.ashemery.com/dfir.html). I am told that this challenge has remained unsolved until now, and I am happy to announce and release how I completed it! It was quite interesting because of the small amount of evidence that was given, and a deep dive needed to be taken to reveal exactly how the attacker gained access to the machine. This post will cover my several hours of digging, and I hope you learn something along the way! Let's go.

As for some background for this challenge: We are tasked with performing an IR investigation from a user who reported they've found a suspicious note on their system. We are given only the contents of the message (seen below) without its file path, and also no time at which the note was left. The evidence is given as a ~25GB E01 image. To start the investigation, I found the suspicious file located at `C:\Tools\README.txt`.

![]("/images/2019-01-02-ali-hadi-forensic-challenge-3-solved/image1.png")

Since we can understand that this note was of concern to the user, it is very important to start developing a time frame of before the note was created to understand what led to this point. This will allow the investigator to find the root cause efficiently.

From here it was decided to start pulling some key artifacts to start analyzing. I first pulled the common registry files including `SAM`, `SOFTWARE`, `SYSTEM`, and `SECURITY`. When viewing the SAM file I found there were only two users on the machine that had previously logged in (master and Administrator). With this being said, each user's `NTUSER` and `UsrClass` files were extracted. Event logs were also extracted, which were eventually used to confirm some actions and events that took place.

I like to familiarize myself with the evidence before digging in too deep. In this case, I noted some important information that I thought would be useful. Some notes include:

![]("/images/2019-01-02-ali-hadi-forensic-challenge-3-solved/image2.png")

As I started to go along, a small timeline was developing. Since I believe this dataset was generated for demonstration purposes, I now understood that we will likely find how the attacker got into the system by focusing from 7:03:15 PM to 7:24:39 PM on 12/11/15 (Time between system install date and "README" created time - this would be unrealistic if given a larger time period.).

To dig into some of the main artifacts, I started to analyze Shellbags from each user to understand some basic every-day actions that may have been performed. This will identify any possible notable locations where the user been when using File Explorer, which may even include USB devices that may have been plugged in, or even zip files that could've been opened. An additional note was found at `C:\Users\master\Desktop\Docs\README.txt` as a result of looking at each user's visited locations. This added to the timeline further of more suspicious activity.

![]("/images/2019-01-02-ali-hadi-forensic-challenge-3-solved/image3.png")

With minimal signs of how the attacker got into the system, looking for suspicious executions was next. When looking for traces of execution, I often use an outwards-in approach by viewing system-wide artifacts first, and then attributing them to a specific user. I started out by looking at the Shimcache. After loading the SYSTEM hive into [Eric Zimmerman's Registry Explorer](https://ericzimmerman.github.io/#!index.md) (thanks Eric!), sorting the entries by time, and focusing in on the narrowed-down time period, the following entries were listed:

![]("/images/2019-01-02-ali-hadi-forensic-challenge-3-solved/image4.png")

When sorting through these logically at first, there was nothing suspicious that I noticed here. The first four entries look to be running the common tools one would see with VirtualBox's Guest Additions. The fifth item in the list is Magnify.exe, the program launched when using the [Magnifier Accessibility Feature](https://support.microsoft.com/en-us/help/11542/windows-use-magnifier-to-make-things-easier-to-see). At first this file does not seem suspicious. However, when analyzing the Shimcache, it is important to understand what can be parsed. Part of each entry in the Shimcache contains the $STANDARD_INFORMATION last modified time of the target file ([ref](https://www.fireeye.com/blog/threat-research/2015/06/caching_out_the_val.html)). When looking at Magnify, it shows this timestamp as 12/11/2015 7:18:54 PM. This time aligns perfectly within our suspected timeline as mentioned above -- which is suspicious. This program is never modified through a Windows installation, but in this case it was modified. Worth a further look? I think so!

![]("/images/2019-01-02-ali-hadi-forensic-challenge-3-solved/image5.png")

At first I thought this was going to be another investigative rabbit hole, so I extracted this file and put it into a virtual machine for further analysis. I generated an MD5 hash of *Magnify.exe* and ran it through VirusTotal to just get an idea of what I was looking at. Without having to upload the program to VT, it looks like it already scanned in the past, and returned clean results. However, the most interesting part of my results was the fact that this came back with a filename of *cmd.exe*. So at this point, I realized exactly what was going on. Magnify was a covert cmd executable!

![]("/images/2019-01-02-ali-hadi-forensic-challenge-3-solved/image6.png")

This file was placed to easily spawn a cmd prompt by using the hotkey from the Windows Magnifier hotkey (Ctrl and +). The attacker placed this file to be able to gain access to a shell from the lock screen or from any place on the computer with local administrator access. This is likely how there was unauthorized access to the machine without leaving traces of some of the common execution artifacts. If a command prompt can be launched at the lock screen, it will not be attributed to a user, therefore those artifacts will not be left on the machine.

After replicating this within my own virtual environment, I was able to swap Magnify.exe for cmd.exe and perform actions with an **system level prompt from the lock screen at anytime**. This is pretty scary, and definitely worth nothing for future investigations. (After a few attempts, interestingly enough Windows Defender picked up on this access and eventually removed Magnify.exe). But **SYSTEM** level access? Wow.

![]("/images/2019-01-02-ali-hadi-forensic-challenge-3-solved/image7.png")

So... how did the attacker gain access? After replicating this within my own environment, the only way this could be done is by changing the owner of the *Magnify.exe* executable and swapping it out with another file of choice. In this specific case, I am unsure if the file was swapped out while the machine was off (files may have been accessed while the machine was off). However, one contradiction is evidence of an INDX record with the filename "cmd.exe", which means cmd.exe was copied and likely renamed to Magnify (maybe the machine was running). **Either way, physical access looks to be the final conclusion. However, I believe this can attack can be accomplished in a few other ways.**

Thank you for spending the time to read about my work, I really hope that this clarifies any confusion for anyone else who takes a stab at solving this case. I also covered a guide of this challenge in [video format](https://www.youtube.com/watch?v=-QZe9p5zbLs&t=8s) if you'd like to check it out!

Stay tuned for more soon!
Adam Ferrante
@ferran7e

Alternate solutions:
* [Harlan Carvey's Writeup](http://windowsir.blogspot.com/2019/01/mystery-hacked-system.html)