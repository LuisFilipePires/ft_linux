# ft_linux
Linux From Scratch

---

## 1. Environment

An already installed Linux virtual machine was used as the base environment for this project.

This was required because the host machine runs on a Mac M1 (ARM architecture), which is not directly compatible with the Linux From Scratch build requirements.

The VM provides a stable x86_64 Linux environment suitable for compiling the entire system from source.

---

## 2. Disk layout

The system was inspected using:

```bash
lsblk

loop20                      7:20   0   221M  1 loop /snap/thunderbird/1045
sr0                        11:0    1  1024M  0 rom  
vda                       253:0    0    64G  0 disk 
├─vda1                    253:1    0     1G  0 part /boot/efi
├─vda2                    253:2    0     2G  0 part /boot
└─vda3                    253:3    0  60.9G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0  60.9G  0 lvm  /
vdb                       253:16   0    10G  0 disk 
vdc                       253:32   0    25G  0 disk
```

## Installation target

### The Linux From Scratch system was installed on:
/dev/vdc

## 3. Disk preparation

### 
Before starting the installation, the target disk was prepared.

bash
`sudo fdisk /dev/vdc`

Created partitions (example structure):

/dev/vdc1 → Linux filesystem (root LFS partition)
/dev/vdc2 → swap (optional, depending on system needs)

## 4. Filesystem creation

#### Formatted the main partition:

`mkfs.ext4 /dev/vdc1`

---

## 5. Mount point setup

### Mount the LFS partition:

```bash
export LFS=/mnt/lfs
mkdir -pv $LFS
mount /dev/vdc1 $LFS
```

## 6. Toolchain preparation

### A temporary toolchain was built inside the LFS environment to isolate the final system from the host system.
This includes:

binutils
gcc
glibc
linux headers
coreutils

## 7. LFS build process

### The system is built step by step following the Linux From Scratch book:

Preparing environment
Building temporary toolchain
Entering chroot
Building final system
Kernel compilation
Bootloader setup (GRUB)
Final configuration

## 8. Notes
All builds were done manually from source.
The system was installed on /dev/vdc.
VM was required due to ARM host limitations.
```
```
```
```
## PROCESS
## 3

```bash 
fill@luisx:~$ sudo fdisk /dev/vdc
[sudo] password for fill: 

Welcome to fdisk (util-linux 2.39.3).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0x3d43163c.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty MBR (DOS) partition table
   s   create a new empty Sun partition table


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-52428799, default 2048): 

Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-52428799, default 52428799): +20G

Created a new partition 1 of type 'Linux' and of size 20 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
## 4 Formating
`sudo mkfs.ext4 /dev/vdc1`

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

---
## Filesystem preparation

The LFS target partition `/dev/vdc1` was successfully created and mounted on `/mnt/lfs`.

The environment variable `LFS` was defined to point to the installation directory:

```bash
export LFS=/mnt/lfs
### The partition was mounted and verified using:

df -h | grep lfs
### Permissions setup
```
The default file creation mask was configured using:

`umask 022`
