---
title: "Phoenix Challenges - Getting Set Up"
date: 2022-05-15T20:00:00-00:00
categories:
  - CTF
  - Phoenix
tags:
  - Phoenix
  - CTF
---

This is the first of a series of writeups for the [Phoenix challenges](https://exploit.education/phoenix/) and explains how those wanting to follow along can get set up. While the binaries are available for the AMD64, ARM64, MIPS64EL, and PPC64EL architectures, my solutions will use the amd64 architecture out of personal convenience. This is because the Ubuntu 64 VM I used in the past for CTF challenges supports it.
{: style="text-align: justify;"} 

<div style="text-align:center">
  <img src="/assets/images/post_images/Phoenix/phoenix.png">
</div>

# What Is Phoenix?
Phoenix is _Exploit.Education’s_ next generation of hacking challenges for teaching “basic memory corruption issues such as buffer overflows, format strings and heap exploitation under \[an\] ‘old-style’ Linux system that does not have any form of modern exploit mitigation systems enabled."[^1] It is the replacement for [Protostar](https://exploit.education/protostar/), the original challenge suite.
{: style="text-align: justify;"} 

# Setting Up
There are two options:

**1\.** **Downloading QEMU Images**<br/>
[QEMU](https://www.qemu.org) is an open-source emulator for virtual machines that runs in the command line. The emulator can be installed onto one's computer. From there, one can download [the QEMU images](https://exploit.education/downloads/) of a preferred architecture and execute a few command-line prompts for the VM to start running! Regarding the process' specifics, I will refer readers to Andrew Lamarra’s [fantastic blog post.](https://blog.lamarranet.com/index.php/exploit-education-phoenix-setup/#Running_the_Phoenix_Image)<br/><br/>The QEMU images offer simplicity and convenience due to the [GEF](https://github.com/hugsy/gef) gdb debugging plugin, [pwntools](https://github.com/Gallopsled/pwntools) for exploit scripting, and [radare2](https://rada.re/r/) for reverse engineering binaries coming pre-installed.
{: style="text-align: justify;"} 

**2\.** **Installing Debian Packages**<br/>
Debian archive packages of the *.deb* extension contain “executable files, libraries, and documentation associated with a particular suite of program or set of related programs.”[^2] They provide a convenient method of installing challenge binaries onto Debian-based systems like Ubuntu. Those interested can go to Phoenix’s [*Downloads* page](https://exploit.education/downloads/), find a package corresponding to the desired architecture, save it to their Debian-based machines, and then double-click it to start installation.<br/><br/>This method offers simplicity of installation for those already having a Debian-based machine, whether virtualized or not. It also provides a greater degree of flexibility for those wanting to tweak their system's settings and install other packages to their liking. From personal experience, trying the same on the provided QEMU images can become a challenge.<br/><br/>I personally had an Ubuntu 22.04 64-bit virtual machine for working with CTF challenges with [pwndbg](https://github.com/pwndbg/pwndbg), [pwntools](https://github.com/Gallopsled/pwntools), and [one_gadget](https://github.com/david942j/one_gadget) installed already, so that is what I went with. Those following in my footsteps should install the same.
{: style="text-align: justify;"} 

---

Regardless of the setup method used, the files to be exploited will be found in the `/opt/phoenix/<architecture>` directories (e.g. amd64, i486, arm64, amd, etc).
{: style="text-align: justify;"} 

Ghidra, an NSA-developed tool for reverse engineering, analyzing, and decompiling binaries, may also prove useful. It does not need to be installed in a virtual machine specifically because it performs static analysis – i.e. the binaries are <u>not executed during analysis.</u>
{: style="text-align: justify;"} 

With setup complete, we are ready for the first challenge. 
{: style="text-align: justify;"}

[^1]: https://exploit.education
[^2]: https://www.debian.org/doc/manuals/debian-faq/pkg-basics.en.html