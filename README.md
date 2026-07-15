# ft_linux
Linux From Scratch

---

## 1. Environment

A Linux Debien virtual machine was used as the host environment for this project.

An additional 10GB virtual disk was attached to the VM to store the LFS system.

the enunced
- You must use at least 3 different partitions: `root, /boot and a swap` partition. You
can, of course, make more partitions if you want to.

---

Disk setup

The system uses one additional disk (/dev/vdb) with 25GB allocated and divided into 3 partitions as required by the subject:

/dev/vdb1 → 2GB → swap
/dev/vdb2 → 1GB → /boot
/dev/vdb3 → ~22GB → root filesystem (/)

This satisfies the requirement of at least 3 partitions.

---

## Partition tools

`lsblk` - list block devices

`fdisk` - manipulate disk partition table

- add partition

sudo fdisk /dev/vdb

Commands:

n → new partition
p → primary
d → delete partition
w → write changes

2 GB -> for swap, 1 GB for boot -> rest OS

- to delete a partition

sudo fdisk /dev/vdb
p - partition
d - delete
w - write -> save

---

- before
```bash
lsblk

vda                       253:0    0    64G  0 disk 
├─vda1                    253:1    0     1G  0 part /boot/efi
├─vda2                    253:2    0     2G  0 part /boot
└─vda3                    253:3    0  60.9G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0  60.9G  0 lvm  /
vdb                       253:16   0    25G  0 disk 
```

- After
``` bash
vda                       253:0    0    64G  0 disk 
├─vda1                    253:1    0     1G  0 part /boot/efi
├─vda2                    253:2    0     2G  0 part /boot
└─vda3                    253:3    0  60.9G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0  60.9G  0 lvm  /
vdb                       253:16   0    25G  0 disk 
├─vdb1                    253:17   0     2G  0 part 
├─vdb2                    253:18   0     1G  0 part 
└─vdb3                    253:19   0    22G  0 part 
```
## Next step
## Filesystem setup
```bash
sudo mkswap /dev/vdb1
sudo swapon /dev/vdb1

sudo mkfs.ext4 /dev/vdb2
sudo mkfs.ext4 /dev/vdb3
```

- ## Mounting LFS system
```bash
export LFS=/mnt/lfs
sudo mkdir -pv $LFS
sudo mount /dev/vdb3 $LFS

sudo mkdir -pv $LFS/boot
sudo mount /dev/vdb2 $LFS/boot
```

---

- ## LFS environment variable

$LFS defines the root directory of the Linux From Scratch system.

It is used to avoid hardcoding paths and to clearly separate the host system from the LFS build environment.

/dev/vdb3 is mounted at /mnt/lfs

$LFS points to /mnt/lfs

- ## LFS user setup

sudo groupadd lfs

sudo useradd -s /bin/bash -g lfs -m -k /dev/null lfs

sudo passwd lfs

Switch to the build user:

su - lfs

export LFS=/mnt/lfs


- ## Sources directory
sudo mkdir -pv $LFS/sources

sudo chmod -v a+wt $LFS/sources

