---
layout: post
title: Ali Hadi's Challenge #3 | Solved!
categories: [Writeup, Forensics]
---

Today I'll be discussing and walking through my solution to Ali Hadi's "Mystery Hacked System" challenge located on his website, [here](https://www.ashemery.com/dfir.html). I am told that this challenge has remained unsolved until now, and I am happy to announce and release how I completed it! It was quite interesting because of the small amount of evidence that was given, and a deep dive needed to be taken to reveal exactly how the attacker gained access to the machine. This post will cover my several hours of digging, and I hope you learn something along the way! Let's go.

As for some background for this challenge: We are tasked with performing an IR investigation from a user who reported they've found a suspicious note on their system. We are given only the contents of the message (seen below) without its file path, and also no time at which the note was left. The evidence is given as a ~25GB E01 image. To start the investigation, I found the suspicious file located at `C:\Tools\README.txt`.