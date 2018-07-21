---
layout: post
title: "Defusing a Binary Bomb: Part 1"
date: "2018-01-26"
slug: "binery_bomb_1"
description: "Reverse engineer the legendary bomb labs using  radare2. By defusing the bomb, the goal is to refine my reversing skills and getting familiar with radare2"
category:
  - views
  - featured
# tags will also be used as html meta keywords.
tags:
  - security
  - reverse engineering
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

If you are trying to get into software security, the are some topics that would give you a strong grasp before diving into malware analysis and binary exploitation. Not to say you cannot learn the latter without them, but it makes it much easier to pickup those topics with a good grasp of the fundamentals.


Having worked with extremely sharp reverse engineers, when asked how does one get started in reverse engineering, the answers can almost always be summarized to these points.


- Understand low level C and how it interacts (Computer Systems)
- Strong operating systems fundamentals
- Basic Compilers
- Read Linkers and Loaders by John R. Levine[^1]

This post is going to help cover a subset of the first bullet point. I would be using the legendary Bomb Lab assignment from the book Computer Systems[^2]. I am by no means a professional reverse engineer, well... at least at the time of this writing. If inevitably make mistakes, fill free to chime in at the comment section below.

## Background

If you are not already familiar with the CPU stack or want to get a good understanding of how the stack works,
Below is a really good series of videos that explains the concepts very well.

{% include youtube.html id="7dLZRMDcY6c" %}

## Tools

I am going to be using radare2. I am new to radare2 so these series of tutorials should help me familiarize myself with it. If you would like to learn more about radare2, checkout this youtube video below on reverse engineering with radare2.

{% include youtube.html id="LAkYW5ixvhg" %}

## Let's Reverse!

Since the is some validation involved, I am guessing that somewhere within the code, the exist a function or an `if` statement that compares my given value with that of the bomb's defuse code.  Lets start by creating a file that would store our defuse codes. We are going to be storing all of our passwords in this file and launch radare2 in debug mode.

```
$ echo "someGuessPassword" > myInputs.txt
$ r2 -d bomb myInputs.txt
```

You should now see the radare2 prompt. To get a basic overview of radare2's debug mode, please reference: [Basic Debugger Screen](https://radare.gitbooks.io/radare2book/content/introduction/basic_debugger_session.html).

We first analyze the binary using `aaa` and place a breakpoint at the main function using `db sym.main`.

> Tip: You can type `db sym.` then TAB key to view some of binary functions.

Enter visual debug mode with `V!`.

```
[radare2Prompt]> aaa
....
[radare2Prompt]> db sym.main
[radare2Prompt]> dc
....
[radare2Prompt]> V!
```

{:.text-center img}
[![Alt text](https://i.imgur.com/1lynF3Ll.png)](https://i.imgur.com/1lynF3L.png)

While in visual mode click `?`, take a moment to view the options then click `q` to escape. We are going to be focus on the windows of the Stack, Registers and Disassembly.

Though better than using gdb, it is still difficult to understand what is going on in the window above if you are new to this. Let's view the execution graph by typing `V`. Let's also turn on the addresses of each instructions using `p`. Take some time, navigate through the graph view, look at the code execution paths.

{:.text-center img}
<a href="https://i.imgur.com/1KYRWOl.png">
  <img src="https://imgur.com/1KYRWOll.png" />
</a>

Lets step into the code with `S` (_Step Over_) until the bomb explodes. As we do this, pay attention to the different instructions that are being called shortly before the bomb explodes.


{:.text-center img}
<a href="https://i.imgur.com/E4tbylF.png">
  <img src="https://imgur.com/E4tbylFl.png" />
</a>

Ahh, there! Notice from the message below that the bomb explodes after the `phase_1` function was called. Now lets add a break point to `phase_1` function. Click `q` until you get to the radere2 prompt. Set a breakpoint at `phase_1`, remove the old breakpoint restart the binary with `do` and lets head back to visual debug mode.

```
[radare2Prompt]> db sym.phase_1
[radare2Prompt]> db -0x080489b0
[radare2Prompt]> do
...
[radare2Prompt]> dc
...
[radare2Prompt]> V!
```

{:.text-center img}
<a href="https://i.imgur.com/i7suqNe.png">
  <img src="https://imgur.com/i7suqNel.png" />
</a>

Now we are getting somewhere interesting... As shown above, two values are pushed to the stack before calling the `strings_not_equal` method. If a function takes in parameters, the parameters are pushed to the stack before the function is called so the registers can easily reference those parameters later using the base pointer. You can actualy verify this using [Compiler Explorer](https://godbolt.org/). As shown below, the values are being pushed to the stack in lines 12,13 before calling the sum function.


<a href="https://i.imgur.com/Zzlulfw.png">
  <img src="https://imgur.com/Zzlulfwl.png" />
</a>

I am guessing here that the `string_not_equal` function is used to compare the value of the string we give the bomb and the actual bomb defuse string. As shown in the previous radare2 screenshot, the is a memory address being passed to the `string_not_equal` function. Radare2 is nice enough to give us the value of the string at that memory address.

Let's confirm our hypothesis. Keep stepping to the next instruction using `S` until we are at the line where `strings_not_equal` is called. The location of the defused string and our guessed answer should have been pushed to the stack. We can actually see this value at the stack pane but it's in Little Endian[^3] so it tedious to read it backwords. Type `:` (_enter command mode_). We can now print the last 2 word sizes that where pushed on the stack using `x/2xw @esp`.

> Word size refers to the maximum amount of bits a register can hold. In our case, we are running a 32 bit program so the word size is 32 bit. If it were a 64 bit program, it would be using the 64 bit registers and a word size would be 64 bits.

<a href="https://i.imgur.com/hCIruus.png">
  <img src="https://imgur.com/hCIruusl.png" />
</a>

Now we can look at the contents of the second memory address that was pushed to the stack. Type `x @0x0804b680` while in command monde.

{:.text-center img}
<a href="https://i.imgur.com/i7suqNe.png">
<a href="https://i.imgur.com/2JgQKFA.png">
  <img src="https://imgur.com/2JgQKFAl.png" />
</a>

Awesome! that was our first input to the bomb program. Now Lets checkout what is located at the memory address that was passed to the first parameter. Type `x @0x080497c0` while in command monde.

{:.text-center img}
<a href="https://i.imgur.com/uZ02Qyl.png">
  <img src="https://imgur.com/uZ02Qyll.png" />
</a>

Bullseye! now lets try defusing the bomb.

```
$ echo "Public speaking is very easy." > myInputs.txt
$ ./bomb myInputs.txt
```
<a href="https://i.imgur.com/8EEhjXp.png">
  <img src="https://imgur.com/8EEhjXpl.png" />
</a>


Awesome, 1 down 5 to go!

### References

[^1]: [Linkers and Loaders by John R. Levine](https://www.amazon.com/Linkers-Kaufmann-Software-Engineering-Programming/dp/1558604960)
[^2]: [Computer Systems: A Programmer's Perspective (2nd Edition) 2nd Edition](https://www.amazon.com/Computer-Systems-Programmers-Perspective-2nd/dp/0136108040/ref=asap_bc?ie=UTF8)
[^3]: [Big-little endian](https://en.wikipedia.org/wiki/File:Big-little_endian.png)
