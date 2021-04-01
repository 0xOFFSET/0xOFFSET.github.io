---
title: Boot process in Linux
tags: Linux
---

Boot process is responsible for firing the Operating System from powering on until loading the mandatory processes to pass control to the user.

## 1- BIOS :
(Basic Input Output System) is a firmware stored in non-volatile chip on motherboard. It has two jobs ;
* Checking and configuring the attached hardware relying on a process called POST — power on self test, and gives alarms if there’s any problem with these resources.
* Searching for something called BootLoader , loading and executing it.

## 2- MBR / GPT :
Are two ways of storing partition information on a drive.

### 2.1 MBR :
(Master Boot Record) is a special boot sector at the beginning of a drive. It contains the Bootloader for installed OS and info about partitions.
Partitioning and boot data are stored in one place.

### 2.2 GPT :
(GUID Partition Table)
It forms a part of something called EFI/UEFI (Extensible Firmware Interface) — a new replacement for older BIOS.
There is no limits on partitions number. Unlike MBR which can handle 4 primary partition at maximum, it handles up to 128 partitions on Windows.
GPT style partitions have CRC (Cyclic Redundancy Check ) to check that data is intact (not corrupted) periodically.
It stores multiple copies of its data across the disk.


## 3- Bootloader :
A small program that places OS into memory (RAM) , stored in chip or came with the OS.
There are two steps that bootloader passes through.

#### 1. First stage :
For MBR/BIOS style , it examines the partition table for bootable partition then run something called GRUB (Grand Unified Bootloader) — the default bootloader for linux.
For modern EFI/UEFI , it reads boot manager data to determine which UEFI application to launch. Then , GRUB is executed.

#### 2. Second stage :
Bootloader resides under directory(folder) called /boot, then a splash screen appears to check which OS to run (if there are multiple OS).
As a final step , the kernel is compressed and loaded into RAM .

## 4- Initial RAM Disk :
a space on disk contains important programs and binary files, stored by initramfs — an initial filesystem responsible for :
* Mounting proper root filesystem
* Providing some kernel functionalities
* Locating devices
* Locating drivers and loading them
* Checking for errors in root filesystem

Eventually, after doing its jobs , it’s cleared from RAM and main process is run.

## 5- Main process :
Historically , linux has (init) as main process, which is keeping system running and working as a manager for all non-kernel process.
When system is up , it runs its sub-processes and responsible for starting or killing these startup processes.

#### Startup system :
Initially , linux has (SysVinit) as default startup system . But because it viewed things as serial process , it did not make use of parallel processing.
(systemd) came as a replacement for the latter . It replaces a serialized set of steps with parallelization techniques .
* It provides fast booting
* Complicated startup scripts in older SysVinit is replaced by simple configuration files.

## 6- GUI :
As a final step , man process runs Graphical User Interface services — if there are enabled .
GUIs consist of there components:

#### 6.1 X Windows System :
provides the basic framework for GUI environment like : interacting with keyboard and mouse , screen display and input/output operations to and from screen.

#### 6.2 Desktop Environment :
is an implementation of a bundle of programs running on the top of computer sharing common GUI schemes like : icons, windows, toolbars, or several widgets.
Examples of Common linux Desktop Environment :

* GNOME
* UNITY
* KDE
* XFCE and LXDE (light weight)

#### 6.3 Display Manager :
a service running in background responsible for :
* Display Management
* Load X Server
* Manage Graphical Logins

Some examples of display manager :
* gdm ~ GNOME
* lightdm ~ Unity
* kdm ~ KDE

## What if GUI hangs ?
Easily , press CTRL+ALT+(F1 .. F6) .
A black screen appears , asks for user logins and open a session to the shell . That is Virtual Terminal .
There can be only one opened VT at one time on screen . GUI services takes one VT space from (F1) to (F6).