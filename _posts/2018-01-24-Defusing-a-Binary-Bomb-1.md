---
layout: post
title: "Defusing a Binary Bomb: Lab 1"
date: "2018-01-26"
slug: "cis194_week_1"
description: "Learn reverse engineering by defusing the legndary bomb labs using gdb and radare2. By defusing the bomb, you would learn the basics of reverse engineering and how the computer executes instructions."
category: 
  - views
  - featured
# tags will also be used as html meta keywords.
tags:
  - security
  - ctf
show_meta: true
comments: true
mathjax: true
gistembed: true
published: true
noindex: false
nofollow: false
# hide QR code, permalink block while printing.
hide_printmsg: false
# show post summary or full post in RSS feed.
summaryfeed: false
## for twitter summary card with squared image and page description or page excerpt:
# imagesummary: foo.png
## for twitter card with large image:
## for twitter video card: (active for this page**
---

## Strong fundamentals

If you are trying to get into software security, the are some topics that would give you a strong grasp before diving into malware and binary exploitation. Not to say you cannot learn the ladder without them, but it just makes it much easier to pickup those topics with a good grasp of the fundamentals. 


Having worked with extremely sharp reverse engineers, when asked how does one get started in reverse engineering, the ansers can almost always be summarized to these points. 


- Understand low level C and how it interacts (Computer Systems)
- Strong operating systems fundamentals
- Basic Compilers
- Read Linkers and Loaders by John R. Levine[^1]

This post is going to help cover a subset of the first bullet point. I would be using the legendary Bomb Lab assignment from the book Computer Systems. I am by no means a professional reverse engineer, well... at least at the time of this writing. If I do inevitable make mistakes, fill free to chime in at the comment section below. 

## Background

If you are not already familiar with the stack and If you want to get a good understanding of how the stack works, 
Below is a really good series of videos to help understand 

## Tools

I am using GDB and Radare2 to view the execution flow graph of the binary. My GDP is have been customized with using GDP dashboard.

## Let's Reverse!

To validate 

### References

[^1]: [Linkers and Loaders by John R. Levine](https://www.amazon.com/Linkers-Kaufmann-Software-Engineering-Programming/dp/1558604960)


