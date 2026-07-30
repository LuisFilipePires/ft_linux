# ft_linux


## Linux From Scratch

Official documentation:

[Linux From Scratch 12.3 Book](https://www.linuxfromscratch.org/lfs/view/12.3/)

---

# Re-enter the LFS Chroot Environment

After rebooting the host system, follow these steps to enter the LFS **chroot** environment and continue building Linux From Scratch.

## 1. Become the root user

```bash
su -
```

Verify that you are root and that the `LFS` environment variable is set:

```bash
whoami
echo $LFS
```

Expected output:

```text
root
/mnt/lfs
```

---

## 2. Verify that the LFS partition is mounted

```bash
findmnt $LFS
```

Expected output:

```text
/mnt/lfs   /dev/sdb2
```

If it is not mounted:

```bash
mount /dev/sdb2 $LFS
```

---

## 3. Create the required mount points

Run this command even if the directories already exist:

```bash
mkdir -pv $LFS/{dev,proc,sys,run}
```

---

## 4. Mount the virtual kernel filesystems

```bash
mount --bind /dev $LFS/dev

mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts

mount -vt proc proc $LFS/proc

mount -vt sysfs sysfs $LFS/sys

mount -vt tmpfs tmpfs $LFS/run
```

Mount `/dev/shm`:

```bash
if [ -h $LFS/dev/shm ]; then
    install -v -d -m 1777 $LFS$(realpath /dev/shm)
else
    mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi
```

---

## 5. Enter the chroot environment

```bash
chroot "$LFS" /usr/bin/env -i \
    HOME=/root \
    TERM="$TERM" \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin \
    MAKEFLAGS="-j$(nproc)" \
    TESTSUITEFLAGS="-j$(nproc)" \
    /bin/bash --login
```

---

## 6. Verify the chroot environment

```bash
whoami
pwd
```

Expected output:

```text
root
/
```

If the output matches the above, the chroot environment is ready and you can continue with the next LFS package.


<details>

<summary>Requirements</summary>

## Host System Kernel

LFS requires that the host kernel supports UNIX 98 pseudo terminals (PTYs).

### Check the kernel version

```bash
uname -r
```

Output:

```text
6.1.0-51-amd64
```

The Linux 6.1 kernel is newer than the minimum required version (Linux 5.4).

### Verify UNIX 98 PTY support

```bash
grep CONFIG_UNIX98_PTYS /boot/config-$(uname -r)
```

Expected output:

```text
CONFIG_UNIX98_PTYS=y
```

</details>

---

## Disk Management

<details>
<summary>Create Virtual Machine</summary>

A Debian Linux virtual machine (40 GB) was used as the host environment for this project.

An additional virtual disk was created to store the LFS system.

**VirtualBox**

`Settings → Storage → SATA Controller → Add Hard Disk (+) → Create`

- Name: `ft_linux.vdi`
- Size: `25 GB`
- Type: `VDI`
- Storage: `Dynamically allocated`

---

## Project Requirement

The project requires at least three partitions:

- `/`
- `/boot`
- `swap`

---

## Disk Setup

### The `/dev` directory

`/dev` (device) is a special directory where Linux represents hardware devices as files.

Storage devices and their partitions are accessed through this directory.

Examples:

- `/dev/vda` → first disk
- `/dev/vda1` → first partition of the first disk
- `/dev/vdb` → second disk
- `/dev/vdb1` → first partition of the second disk

For this project, the additional disk (`/dev/vdb`) is divided into three partitions:

| Partition | Size | Purpose |
|-----------|------|---------|
| `/dev/vdb1` | 2 GB | Swap |
| `/dev/vdb2` | 1 GB | `/boot` |
| `/dev/vdb3` | Remaining space | Root filesystem (`/`) |

This satisfies the project requirement.

---

## Partitioning Tools

- `lsblk` — list block devices
- `fdisk` — create and manage disk partitions

---

## Creating the Partitions

```bash
sudo fdisk /dev/vdb
```

Main commands used inside `fdisk`:

- `n` → create a new partition
- `p` → print the partition table
- `d` → delete a partition
- `w` → write changes and exit

Partition layout:

- 2 GB → swap
- 1 GB → /boot
- Remaining space → root filesystem

---

## Verify the Partition Layout

### Before

```bash
lsblk
```

```text
vda                       253:0    0    64G  0 disk
├─vda1                    253:1    0     1G  0 part /boot/efi
├─vda2                    253:2    0     2G  0 part /boot
└─vda3                    253:3    0  60.9G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0  60.9G  0 lvm  /
vdb                       253:16   0    25G  0 disk
```

### After

```text
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

---

## Creating the Filesystems

Initialize the swap partition:

```bash
sudo mkswap /dev/vdb1
sudo swapon /dev/vdb1
```

Create the filesystems:

```bash
sudo mkfs.ext4 /dev/vdb2
sudo mkfs.ext4 /dev/vdb3
```

---

## Mounting the LFS System

```bash
export LFS=/mnt/lfs

sudo mkdir -pv $LFS
sudo mount /dev/vdb3 $LFS

sudo mkdir -pv $LFS/boot
sudo mount /dev/vdb2 $LFS/boot
```

---

## The LFS Environment Variable

`$LFS` defines the root directory of the Linux From Scratch system.

It avoids hardcoding paths and clearly separates the host system from the LFS build environment.

In this setup:

- `/dev/vdb3` is mounted at `/mnt/lfs`
- `$LFS` points to `/mnt/lfs`

---

## Creating the LFS User

```bash
sudo groupadd lfs

sudo useradd -s /bin/bash -g lfs -m -k /dev/null lfs

sudo passwd lfs
```

Switch to the build user:

```bash
su - lfs
```

Set the environment variable again:

```bash
export LFS=/mnt/lfs
```

---

## Sources Directory

Create the sources directory:

```bash
sudo mkdir -pv $LFS/sources
```

Grant write permission with the sticky bit:

```bash
sudo chmod -v a+wt $LFS/sources
```

---

## Making the LFS Variable Persistent for Root

The LFS book requires the `LFS` environment variable to be available when working as `root`.

Verify whether it is already defined:

```bash
sudo -i
echo $LFS
```

If no value is displayed, add it to the root user's shell configuration:

```bash
echo 'export LFS=/mnt/lfs' >> ~/.bashrc (debian) -> generic /.bash_profile
source ~/.bash_profile
```

Verify the result:

```bash
echo $LFS
```

Expected output:

```text
/mnt/lfs
```

Exit the root shell:

```bash
exit
```

### Directory permissions

The LFS book requires the sources directory to have specific permissions:

```chmod a+wt $LFS/sources```

Expected result:

```drwxrwxrwt sources```

### sticky bit

The final t represents the sticky bit. It allows multiple users to work inside the directory while preventing users from deleting files owned by other users.

---

---

## Notes

### Resizing a VirtualBox Virtual Disk

List all virtual disks:

```bash
VBoxManage list hdds
```

Locate the disk:

```text
Location: /media/<USER>/SSD/ft_linux_builder_MV2/ft_linux_builder/ft_linux_builder.vdi
```

Resize the disk:

```bash
VBoxManage modifymedium disk "PATH_TO_DISK" --resize SIZE_IN_MB
```

Example:

```bash
VBoxManage modifymedium disk "/media/<USER>/SSD/ft_linux_builder_MV2/ft_linux_builder/ft_linux_builder.vdi" --resize 40960
```

</details>

---

## Packages

<details>
<summary>Packages to Install</summary>

In $LFS/sources/ forder download each package

Using wget https://downlod_file.tar.xz -> links in LFS book chapter 3

```md5sum -c md5sums``` To confirm veracity files

## Storage of Source Packages

During the Linux From Scratch (LFS) build process, all required source packages and patches are stored in the directory:

```bash
$LFS/sources
```

## Packages and patches

```bash
• Acl
• Attr
• Autoconf
• Automake
• Bash
• Bc
• Binutils
• Bison
• Bzip2
• Check
• Coreutils
• DejaGNU
• Diffutils
• Eudev
• E2fsprogs
• Expat
• Expect
• File
• Findutils
• Flex
• Gawk
• GCC
• GDBM
• Gettext
• Glibc
• GMP
• Gperf
• Grep
• Groff
• GRUB
• Gzip
• Iana-Etc
• Inetutils
• Intltool
• IPRoute2
• Kbd
• Kmod
• Less
• Libcap
• Libpipeline
• Libtool
• M4
• Make
• Man-DB
• Man-pages
• MPC
• MPFR
• Ncurses
• Patch
• Perl)
• Pkg-config
• Procps
• Psmisc
• Readline
• Sed
• Shadow
• Sysklogd
• Sysvinit
• Tar
• Tcl
• Texinfo
• Time Zone Data
• Udev-lfs Tarball
• Util-linux
• Vim
• XML::Parser
• Xz Utils
• Zlib
```

**Differences**

*Linux is constantly evolving. Every few months, new releases introduce changes, such as updated packages, renamed components, or different system organization. Therefore, some packages mentioned in older documentation may have different names or implementations in newer versions.*

*For example, older LFS versions used Eudev, while newer LFS versions use udev-lfs, which provides the required device management functionality in a different way.*

|  42 ft_linux   |   LFS 12.3   |
|----------------|--------------|
| Eudev	| udev-lfs|
| Sysvinit | SysV init |

```
During the download phase, Expat 2.6.4 was renamed by the upstream project as:
`expat-2.6.4-RENAMED-VULNERABLE-PLEASE-USE-2.7.2-INSTEAD.tar.xz` 
The file content was verified using the MD5 checksum provided by LFS 12.3.
```

</details>

<details>

<summary>Configuring the `lfs` User Environment</summary>

## Configuring the `lfs` User Environment

After creating the `lfs` user, all temporary system packages are built using this unprivileged account instead of `root`.

### Switch to the `lfs` User

```bash
su - lfs
```

Verify the current user:

```bash
whoami
```

Expected output:

```text
lfs
```

---

## Configure the Bash Environment

A new `~/.bash_profile` was created to start a clean shell environment.

```bash
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

This command removes almost all inherited environment variables from the host system (Debian), preventing them from interfering with the Linux From Scratch build.

After creating the file, it was loaded manually:

```bash
source ~/.bash_profile
```

---

A new `~/.bashrc` file was also created:

```bash
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```

This file defines the environment variables required to build the temporary LFS system.

---

## Disable `/etc/bash.bashrc`

Debian automatically loads the global Bash configuration file:

```text
/etc/bash.bashrc
```

This file may modify the build environment by adding aliases, changing the `PATH`, or exporting additional environment variables.

To ensure a clean and reproducible build environment, the LFS book recommends temporarily renaming this file.

As `root`:

```bash
[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE
```

Output:

```text
renamed '/etc/bash.bashrc' -> '/etc/bash.bashrc.NOUSE'
```

> **Note:** This file can be safely restored after completing the LFS build.

---

## Configure Parallel Compilation

Check the number of available logical processors:

```bash
nproc
```

Output:

```text
3
```

Enable parallel compilation by setting the `MAKEFLAGS` environment variable:

```bash
export MAKEFLAGS=-j3
```

This allows `make` to execute up to three compilation jobs simultaneously, reducing the overall build time.

> **Important:** Never use `make -j` without specifying a number. Doing so allows `make` to create an unlimited number of jobs, which can lead to system instability.
</details>

<details>

<summary>Install Packages</summary>

### Binutils 2.44 (Pass 1)

The first package of the temporary toolchain was successfully built and installed.

Compilation time:
- `make`: approximately **7 minutes**
- `make install`: **7.3 seconds**

The package was installed under:

```text
/mnt/lfs/tools
```

This provides the cross-binutils tools (`ld`, `as`, `ar`, `objdump`, etc.) that will be used to build the rest of the temporary LFS toolchain.

---

## Glibc 2.41

The GNU C Library (glibc) was successfully built and installed into the temporary LFS system.

### Sanity check

The toolchain was verified with:

```bash
echo 'int main(){}' | $LFS_TGT-gcc -xc -
readelf -l a.out | grep ld-linux


---

- ✅ Preparação do disco e montagem de `$LFS`
- ✅ Estrutura de diretórios
- ✅ Utilizador `lfs` e ambiente de compilação
- ✅ Binutils Pass 1
- ✅ GCC Pass 1
- ✅ Linux API Headers
- ✅ Glibc + sanity check

---


</details>


