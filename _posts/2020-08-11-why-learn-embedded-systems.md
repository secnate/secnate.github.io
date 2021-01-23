---
title: "Why Programmers Should Learn Embedded Systems"
date: 2020-08-11T12:34:30-04:00
categories:
  - Coding
tags:
  - Coding

toc: false
--- 

<img src="/assets/images/post_images/08-11-2020/fpga.jpg">

The last semester of my computer science bachelors, I had room for an extra class and took _Embedded Systems._ 
{: style="text-align: justify;"} 

It was fun. It introduced me to the other side of computers, which I didn't learn while tinkering with highly abstracted languages. Over the course of many projects, we configured FGPA boards and programmed them while accounting for memory and processing constraints. For the first time in my life _hardware mattered_.
{: style="text-align: justify;"}

I believe every computer science student and software engineer would benefit from learning embedded systems, even if they won't pursue them as a career. Here's why:
 {: style="text-align: justify;"}

- **Understanding:** Because programming has become highly abstracted, many treat a computer as a box whose contents they don't know and could care less about. Prepackaged libraries allow one to easily draw an image without thinking about how it is loaded from memory, resized, blended with other graphics (if it is not the only one presented), and fed with the VGA protocol into a dual-clock FIFO buffer that the monitor's controller reads. And then there is the process of linking modules and loading the final executable, which involve special steps since much of the target hardware doesn't have an operating system running. Learning how hardware components are designed and programs are deployed for execution strips away the mystery of how a computer works. It is dissapointing that standardized computer science curricula feed a diet of AND, NAND, and OR logic gates in _mandatory_ digital logic classes and don't explain how the hardware they form works.
{: style="text-align: justify;"}

- **Efficiency:** When dealing with limited processing power and cache and memory sizes, tradeoffs are required. Unlike regular computers whose (relatively large) processing capacities allow inefficient algorithms to execute in a required timeframe, embedded systems don't have that luxury. Certain applications such as self-driving cars cannot use the cloud for calculations because the communication latency and potential Internet link issues prevent them from making decisions in the split-seconds they matter. Everything _must_ be done locally.<br/><br/>One has to design hardware and write optimized programs meeting performance requirements that minimize resource consumption. Such optimization skills help with other types of programming.
{: style="text-align: justify;"}

- **IoT:** With the rollout of 5G Internet, Internet-connected devices will transform every sector of the economy. And they are prime examples of embedded systems. Knowing how they work can help engineers understand how the applications and network infrastructure they interact with can be optimized and why. 
{: style="text-align: justify;"}






 
