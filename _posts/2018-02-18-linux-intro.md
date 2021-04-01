---
title: Linux Intro
tags: Linux
---

Mostly any operating system consists of three layers , starting from the nearest to the resources — the kernel . Then, shell and the higher layer — End User Applications.

The first intention to build an operating system was 1964 by MIT, Bell Labs and General Electric corporation.It was called multics.
There were some issues in the latter , so Bell Labs (AT&T nowadays) worked on the project and built UNIX . It was rewritten in C language which made it more portable — working on multiple platform.

Unix was ported into universities , and students created their own tools and programs , then they decided to create their own OS containing these software — called BSD (Berkeley Software Distribution).

Later, because of some business issues , Bell Labs required to license the operating system’s source code to anyone who asked, then Bell Labs began selling UNIX as a proprietary product.
As a result , Ritchard stallman — a member of Bell Labs crew, decided to start:
* GNU project — an OS and extensive collection of computer software
* established FSF (Free software Foundation) and wrote GNU GPL (Global Public License) , trying to save any source code software or OS.

But Ritchard failed to establish a stable OS, or he couldn’t write OS just a collection of tools.

Eventually in 1991 , a student in Helsinki University — Linus Torvalds ,wrote his own kernel (Linux Kernel) which won Ritchard admiration , resulting to adopt it and establish the final product , a stable , multitasking and complete OS — GNU/LINUX. It was licensed under GNU GPL as an open source OS.

Developers and contributors from all over the world began to work on Linux for building their own distributions.

{: style="text-align:center"}
![](/assets/images/linux-intro/OS_arch.jpeg "Operating System Architecture" )
*Operating System Architecture*

### Some Kernel jobs :
* Job scheduling —  controls and organizes CPU times for each task(program).
* Memory management — responsible for allocating tasks in the RAM in certain address and size.
* Storage management — enables the OS to access the storage devices and read or write data.
* Resources management — interacts with attached hardware.

### Distribution:
A full Linux distribution consists of the kernel plus a number of other software tools for file-related operations, user management, and software package management. Each of these tools provides a small part of the complete system.
There are thousands of distros , with three main common families — Debian , RedHat and SUSE.

### Choosing a distro:
* Server or Desktop version ?
* Available space ?
* What about Updates periods ?
* Hardware platform ( ARM, intel, etc..) ?
* Support — you need long term support as an expertise ?
* Packages — depends on your field (Pentesting , AI ,web development ,etc.)
* Kernel Customization — ask the vendor for advanced customization.


