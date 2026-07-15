# ft_linux
Linux From Scratch

---

## Disk Management 

A Linux Debien 40.962 GB virtual machine was used as the host environment for this project.

To store the LFS system, i split 20GB at VirtualBox, ```Debian VM → Settings → Storage → SATA Controller → Add Hard Disk (+) → Create```

- Name: ft_linux.vdi
- Size: 20 GB
- Type: VDI
- Storage: Dynamically allocated

---

## 1 Environment

### The enunced

- You must use at least 3 different partitions: `root, /boot and a swap` partition. You
can, of course, make more partitions if you want to.

---

### Disk setup

### /dev Directory

`/dev` (device) is a special directory in Linux where system devices are represented as files. 
In Linux, hardware devices are accessed through special files called device files, allowing the operating system and applications to communicate with them.

Disks, partitions, USB devices, terminals, and other hardware components are represented inside `/dev`.

For example:
- `/dev/sda` represents a disk device.
- `/dev/sda1` represents the first partition of that disk.
- `/dev/sdb` can represent an additional disk added to the system.

In the ft_linux project, the disk used to build our Linux system appears inside `/dev` (for example `/dev/sdb`). Its partitions are created from this device (for example `/dev/sdb1`) and will be used as the destination for the new Linux system.

The system uses one additional disk (/dev/vdb) with 10GB allocated and divided into 3 partitions as required by the subject:

- /dev/vdb1 → 1GB → swap
- /dev/vdb2 → 1GB → /boot
- /dev/vdb3 → ~18GB → root filesystem (/)

This satisfies the requirement of at least 3 partitions.

---

## Partition tools

`lsblk` - list block devices

`fdisk` - manipulate disk partition table

- add partition

```sudo fdisk /dev/vdb```

Commands:

n → new partition
p → primary
d → delete partition
w → write changes

2 GB -> for swap, 1 GB for boot -> rest OS

- to delete a partition

```sudo fdisk /dev/vdb```

- -p (partition)
- -d (delete)
- -w (write) -> save

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



---
---

## Notes

How to manage space in virtual machines

```bash
VBoxManage list hdds
```

Find the location of HDD to manage

```Location:       /media/"USER"/SSD/ft_linux_builder_MV2/ft_linux_builder/ft_linux_builder.vdi```

```VBoxManage modifymedium disk "PATH" --resize *** ```

Example:

``` VBoxManage modifymedium disk "/media/"USER"/SSD/ft_linux_builder_MV2/ft_linux_builder/ft_linux_builder.vdi" --resize 40960```


