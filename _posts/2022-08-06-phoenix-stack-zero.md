---
title: "Phoenix Challenges - Stack Zero"
date: 2022-08-06T20:00:00-00:00
categories:
  - CTF
  - Phoenix
tags:
  - Phoenix
  - CTF
---

<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/phoenix.png">
</div>

# The Challenge
The challenge’s description and source code are located [here](https://exploit.education/phoenix/stack-zero/). It and all other Phoenix binaries are located in the __/opt/phoenix/amd64__ directory. A [previous blog post](https://secnate.github.io/ctf/phoenix/phoenix-setup/) describes how to set up the Virtual Machine for these challenges, if that hasn't been done already.
{: style="text-align: justify;"} 

# Objective
Looking at _Stack Zero’s_ C code, we see the _changeme_ variable stored in the _locals_ struct initialized to 0. The goal is to tamper with its value and make it non-zero to print the desired statement.
{: style="text-align: justify;"}

# Related Concept
Executed programs look like this inside computer memory:
<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-zero/computer-memory-structure.png">
</div>
The stack and the heap are the two main memory structures. To keep it short: 

- __The Stack__ is used for storing information regarding called functions and their local variables. When a new function is called, the machine expands the stack’s size downwards with more room for storing its local variables and information regarding this function call. Conversely, the machine frees up stack memory and decreases its size upwards when a function is exited. __Stack memory management is automatically performed during execution.__
{: style="text-align: justify;"}

- __The Heap__ is used by programs to allocate memory on the fly during execution. This is especially useful if the amount of memory needed is <u>unpredictable.</u> For example, a browser receiving messages of variable lengths may grab chunks of Heap memory into which incoming messages are to be placed for processing.
{: style="text-align: justify;"}

- The Heap’s memory is managed __explicitly__ by programmers during low-level languages’ execution. The __malloc(), calloc(), free(), and realloc()__ functions are used in C and the __new__ and __delete__ operators are used in C++.
{: style="text-align: justify; list-style-type: none;"}

- Higher-level and memory-safe languages such as Python and Java use memory managers to handle the heap securely <u>out of sight.</u>
{: style="text-align: justify; list-style-type: none;"}

Those desiring further clarification regarding how the Stack and Heap work may find [this explanation](https://courses.engr.illinois.edu/cs225/sp2022/resources/stack-heap/) helpful.
{: style="text-align: justify;"}

# The Bug
All of Stack Zero’s data is stored on the stack, with the _locals_ struct’s _buffer_ and _changeme_ variables being adjacent neighbors. Excess data rammed into the _buffer_ will spill over into the _changeme_ variable and affect its value. This spillover is caused by __gets()__ function for writing console-entered input into the _locals.buffer_ not performing any bounds-checking.
{: style="text-align: justify;"}

# The Exploit
The _locals.buffer_ into which the input is written has space for 64 characters. Since the _locals.changeme_ variable was 0 originally, the exploit needs to tamper with its memory location to make it have a non-zero value. This is done by feeding in an input string of 65 characters, with the last directly spilling over into the _locals.changeme_ variable’s memory and making it non-zero.
{: style="text-align: justify;"}

The exploit was scripted with _pwntools_, a Python framework for exploit development. The code has extensive explanatory comments since it is the first of this series. It can be found in my [Github repository](https://github.com/secnate/Exploit-Education-CTFs) for Phoenix challenge solutions.
{: style="text-align: justify;"}

The result?
```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-zero$ ./stack-zero-exploit.py
Launching The Phoenix Stack Zero Exploit!
[!] Could not find executable 'stack-zero' in $PATH, using '/opt/phoenix/amd64/stack-zero' instead
[+] Starting local process '/opt/phoenix/amd64/stack-zero': pid 12739
[*] Switching to interactive mode
[*] Process '/opt/phoenix/amd64/stack-zero' stopped with exit code 0 (pid 12739)
Welcome to phoenix/stack-zero, brought to you by https://exploit.education
Well done, the 'changeme' variable has been changed!
[*] Got EOF while reading in interactive
$ 
```

# Remediation
To prevent such a memory corruption bug, I would encourage developers to not write in C and to transition to memory-secure languages such as Python or Rust.
{: style="text-align: justify;"}

If there is no choice but to use C, I would caution against using the __[gets()](https://www.tutorialspoint.com/c_standard_library/c_function_gets.htm)__ function to extract inputs from the command line. As just seen, it reads input until the newline or end-of-file characters are reached, regardless of the destination buffer’s size.
{: style="text-align: justify;"}

The __[fgets()](https://cplusplus.com/reference/cstdio/fgets/)__ function should be used instead. It parses command-line input and places it into the destination buffer while performing the appropriate bounds checks.
{: style="text-align: justify;"}

The source code’s `gets(locals.buffer);` line would thus be

`fgets(locals.buffer, 64, stdin);`
{: style="text-align: center;"}

An additional bonus of using __fgets__ is that it automatically terminates the buffer with the terminating _null_ character (“\0”). Programmers may forget to insert such a character manually. So in the case of this challenge, it is only 63 characters that would read from the command line into the buffer, with the 64th being "\0".
{: style="text-align: justify;"}

Terminating a buffer with a null character is critical for preventing buffer over-read vulnerabilities. These involve leaks of data as a function reading a buffer does not meet a terminating character and continues past the buffers’ end into adjacent memory.
{: style="text-align: justify;"}

The most notorious example of such that comes to mind is the [2014 OpenSSL Heartbleed bug.](https://owasp.org/www-community/vulnerabilities/Heartbleed_Bug) For those unfamiliar, OpenSSL is a widely-used open-source implementation of SSL and TLS cryptographic protocols used by websites and systems to establish secure and verified connections. Bruce Schneier, a security expert who has historically been very conservative when talking about the effects of security issues, [called it catastrophic](https://www.schneier.com/blog/archives/2014/04/heartbleed.html):
{: style="text-align: justify;"}
> On the scale of 1 to 10, this is an 11.

---

That’s all for this round. Stay tuned for the _Stack One_ challenge, an extension of _Stack Zero_ -- with a twist!