---
title: "Introduction to ROP Attack"
teaching: 10
exercises: 30
questions:
- "What is ROP Attack?"
- "How to build the payload?"
- "What can users do to prevent ROP from happening"
objectives:
- "Successfully conduct a ROP Attack."
keypoints:
- "User needs to know the vulnerability of the ELF file provided and know how to build the payload"
---

## ROP Attack

- ROP stands for "Return-Oriented Programming" and is a type of attack that allows an attacker to take control of a program's execution flow by manipulating its stack.

- ROP attacks work by finding "gadgets" in the program's code, which are short sequences of instructions that end with a "return" instruction. By chaining these gadgets together, the attacker can construct a new program flow that achieves their desired outcome.

- ROP attacks are often used to bypass security measures like DEP (Data Execution Prevention) and ASLR (Address Space Layout Randomization), which are designed to prevent traditional buffer overflow attacks.

- ROP attacks can be difficult to defend against, as they don't rely on injecting code into the program's memory like traditional attacks do. Instead, they rely on using the program's own code against itself.

### Demonstration

The demonstration on CHEESEHub illustrates the ROP attack using one machines; 
1. We will  analyze the ELF File provided to know what's the vunerabilities of the program 
2. We will build the payload based on that. We verify the success of the attack by printing the flag.txt.

> ## prerequisites
> 
> You will need to conduct the Buffer Overflow attack prior to the ROP attack. If you're not familiar with it, you could refer to [Lesson: Introduction to Buffer Overflow Attack](https://github.com/cheese-hub/BufferOverflowAttack)
> You will need to know the knowledge of [Assembly Language](https://github.com/cheese-hub/ROP-Attack/blob/main/assembly%20language.md), [pwntools](https://github.com/cheese-hub/ROP-Attack/blob/main/pwntools.md), and [Radare2](https://github.com/cheese-hub/ROP-Attack/blob/main/r2.md) prior to the ROP Attack.
{: .prereq} 

> ## Getting Started
> 
> You will need to create an account on [CHEESEHub](https://www.hub.cheesehub.org) to work through this exercise.
> Each container in this demonstration has a web interface and is accessible through your web browser, no other special software 
> is needed.
{: .callout} 

We will start by first adding the SSLStrip application:

![Add SSLStrip]({{ page.root }}/fig/sslstrip/add-sslstrip.png)

Next, click the *View* link to go to the application-specific page and start the application containers:

![Start Containers]({{ page.root }}/fig/sslstrip/start-containers.png)

> ## Container Launch Errors
>
> If a container fails to start, you will see a flashing warning icon next to the container's name. Take a look at the logs to 
> determine the cause of failure and try deleting the application and restarting.
{: .callout}

Once all the containers have started, launch each container's web interface in a separate browser tab by clicking the icon 
next to the container's name.
## Let's get start it



## Analyze the ELF File
Here is the figure of r2 -AAAA split
# Figure of results using li
Check the program's protections and find that NX is enabled.

## Find the vulnerabilities
using afl to list all the functions involved

After analyzing with r2, the main, pwnme, and usefulFunction functions were discovered.
3 figures of find functions

We can see in the pwnme function that there is a buffer with the size of 40 bytes but fgets allow to unput 96 bytes
So an overflow will happen and we can use as the start of the payload.

We can see that the usefulFunction is gonna to call system function with string '/bin/ls' as its parameter
After we run the program, we can see that '/bin/ls' is used.
But our goal is to use  '/bin/cat flag.txt' as its parameter so that we can print the flag. So keep searching the gadgets we want.



Let's get back to r2 and use izz to list the string inside the ELF File.

We can see that there is a string that can print the flag:/bin/cat flag.txt


And the address of this string is 0x0001060.

## The idea came out!
Now we try to make an overflow and return to the address of system function after that and try to exec the system function with the parameter /bin/cat flag.txt



代码生成与分析


## Catch the Flag!!
## Contermeasure?

> ## ROP Split Attack
> 
> In this lesson, the hacker is able to strip the SSL via ARP poisoning the victim. Once the ARP Poisoning is initiated, the hacker is
> able to intercept the traffic from the victim to the server, acting as the server and possibly collecting confidential information from the victim. 
{: .callout}

{% include links.md %}

