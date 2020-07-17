---
title: "Learning to Code For Infosec"
date: 2020-07-14T12:34:30-04:00
categories:
  - Advice
tags:
  - Advice
  - Coding
---

A commonly-repeated mantra for current and future practicioners at InfoSec conference talks, blog posts, and Twitter threads is that one _must learn to code._ Websites, tools, and proof of concept exploits. Those who can't are severely limited career-wise because they remain dependent on those who can.
{: style="text-align: justify;"} 

Great advice. However, its givers typically overlook a key question that receivers -- especially beginners with less knowledge of or are undecided between different [InfoSec domains](https://medium.com/bugbountywriteup/jobs-in-information-security-infosec-93a5efc12ca2) -- usually have:
{: style="text-align: justify;"} 

> What type(s) of coding should I learn to be able to 
> produce strong results while being flexible to 
> pivot across different InfoSec domains?

This overview will address this issue by deliniating a learning pathway.

This post has my recommendations on what one should learn and explains why but _not how._ My [coding resources page](https://secnate.github.io/resources/coding/) covers that in extensive detail.
{: .notice--info style="text-align: justify;"}

# Python

Python should be the first step for beginners. Its minimal syntax and automatic memory management allows beginners to focus on mastering basic concepts without significant distractions.
{: style="text-align: justify;"}

Its ease of scripting combined with powerful standard and third-party libraries makes it well-suited for  performing a broad variety of tasks. Hence, it is the language of choice for making automation and analysis tools for every concievable InfoSec need, ranging from digital forensics and penetration testing to applied data science and open-source intelligence. Even if one doesn't write the tools they use, examining their workings may sometimes be a necessity. Its explosion in popularity among developers means that application security practicioners and bug hunters need to be able to audit ever-expanding Python codebases. Finally, knowing how to work with Python makes for an easy transition to Bash, Powershell, and other scripting languages that system administrators use when working in the blue team.
{: style="text-align: justify;"}

When starting, use it to learn programming basics. What arrays, linked lists, queues, stacks, and heaps are. How class inheritance works. A solid grounding in the programming fundamentals will prepare you well for working with its InfoSec applications. One can't run before learning how to walk.
{: style="text-align: justify;"}

# C++

C++ was released in 1998 as an object-oriented extension of C and has become widely used since. Its backwards compatability with C means that one learns two languages for the price of one!
{: style="text-align: justify;"}

C and C++ are

1. **Memory unsafe:** programmers can directly access and manipulate computer memory.
2. **Everywhere:** operating systems, applications, embedded systems, device drivers, browsers, databases, you name it.

This means that hackers can use their memory-accessing capabilities to attack a broad variety of targets. And while memory-safe alternatives like [Rust](https://en.wikipedia.org/wiki/Rust_(programming_language)) are gaining market share, rewriting million-line codebases is an expensive and time-consuming task few companies want to do. **A perfect storm.**
{: style="text-align: justify;"} 

This has its effects. Reverse engineers need to deduce how an exploit was originally written. Application security teams need to develop and evangelize company-wide standards for secure coding practices. Penetration testers, bug hunters, and exploit developers need to understand how systems can be attacked. Knowing C/C++ and their memory-manipulation capabilities is _essential._
{: style="text-align: justify;"}

An additional benefit of C++ is that it doesn't conceal functionalities from the programmer, unlike other languages. It is like driving a car in manual. Java or C# are like driving with automatic transmission. Learning it helps one to understand what is happening on underneath the proverbial hood and makes transitioning to other languages easy.
{: .notice--info style="text-align: justify;"}

# Assembly

All programs get converted by a [compiler](https://en.wikipedia.org/wiki/Compiler) into it. This is as close as human-readable programs get to computers' natural language; processor hardware directly maps it into binary for execution.
{: style="text-align: justify;"}

Because compiling code from a higher-level language to assembly is an irreversible process, forensics and malware analysts read binaries dissembled into Assembly. Exploit devlopers, penetration testers, and security researchers also use it to uncover vulnerabilities and to craft and modify payloads attacking them.
{: style="text-align: justify;"}

***

This path should fulfill the goals of helping learners master programming concepts in a learner-friendly fashion while equipping them with the skills they need to pivot into any InfoSec (or even tech) domain of their choosing. I hope this helps.
{: style="text-align: justify;"}

