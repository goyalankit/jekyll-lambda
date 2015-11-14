---
layout: post
title: "kernel compilation notes"
date: 2014-09-04 15:57:47 -0500
comments: true
categories:
---

## Kernel Versions
```
# Guest
goyal@fyra:~$ uname -a
Linux fyra 3.13.0-34-generic #60-Ubuntu SMP Wed Aug 13 15:45:27 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

# Host
ankit@ubuntu2:~$ uname -a
Linux ubuntu2 3.16.1 #1 SMP Thu Sep 4 18:57:37 CDT 2014 x86_64 GNU/Linux
```

## Create VM

- Install KVM and it's dependencies.
- Create a Ubuntu VM using VMBuilder

```sh
sudo vmbuilder kvm ubuntu     \
    --arch i386               \
    --user ankit              \
    --name ankit              \
    --pass cracker            \
    --suite lucid             \
    --flavour virtual         \
    --addpkg openssh-server   \
    --libvirt qemu:///system
```

<!-- more -->

### .qcow image
qcow is a file format for disk image files used by QEMU, a hosted virtual machine monitor.
the old QEMU copy-on-write format, supported for historical reasons and superseded by qcow2

### .qcow2 image
QEMU copy-on-write format with a range of special features, including the ability to take multiple snapshots, smaller images on filesystems that don't support sparse files, optional AES encryption, and optional zlib compression

- **Basic Commands**

```sh
  arp -n # to find out what ip address was assined to the VM.

  virsh --connect qemu:///system list --all # to list all the vms

  virsh --connect qemu:///system start ubuntu # to start the vm with name ubuntu
```
- For clean shutdown install **acpid** inside the guest OS. `sudo apt-get install acpid`.

```
virsh --connect qemu:///system shutdown ubuntu # to shutdown the vm
```

`virsh` is a tool that provides interface to libvrt deamon.

`qemu-`

#### Run the Virtual Machine using KVM

```
kvm -drive file=tmpYPpuMh.qcow2 --snapshot
```
### Run the VM using qemu-system

This configuration allows guest to access outside world but not vice-versa. It forwards 22 port so that host can connect to VM over ssh.
```
qemu-system-x86_64 -drive file=tmpPKS3JS.qcow2 --snapshot -net nic,model=virtio -net user -redir tcp:2222::22
```

To connect to guest
```
ssh -p 2222 ankit@localhost
```

### Mount image on local filesystem

#### Network Block Device (nbd)

From Wikipedia:

In Linux, a network block device is a device node whose content is provided by a remote machine. Typically, network block devices are used to access a storage device that does not physically reside in the local machine but on a remote one. As an example, the local machine can access a fixed disk that is attached to another computer.


* Load the **nbd** module in current kernel

```
sudo modprobe nbd max_part=8
```
* create a device entry for the disk
```
sudo qemu-nbd --connect=/dev/nbd0 file.qcow2
```

* Mount the device on mount point
```
sudo mount /dev/nbd0p1 /mnt/kvm
cd /mnt/kvm # should show you the filesystem of VM
```

* Unmount
```
sudo umount /mnt/kvm
sudo nbd-client -d /dev/nbd0
```

---

### Compile the kernel

* Download and extract source from http://kernel.org
* Create a separate kbuild directory.
* Prepare kernel for compilation by cleaning the source directory
```
make mrproper
```
* Create configuration file in kbuild directory
```
yes "" | make -C <kernel dir> O=$(pwd) config
```
* Set `CONFIG_SATA_AHCI=y` in .config file so that SATA disk driver is built into kernel and not as a module. This will allow kernel to boot off a virtual SATA drive without having to load a module.
* Build the kernel by running make in kbuild

---

#### Boot the kernel

* Tell kernel that root of the filesystem is `/dev/sda1`
* Set the console to `ttyS0` so that we can see the logs.

```
sudo qemu-system-x86_64 -drive file=tmpPKS3JS.qcow2 --snapshot -net nic,model=virtio -net user -redir tcp:2222::22 -nographic -kernel ~/workspace/kernel/kbuild/arch/x86_64/boot/bzImage -boot c -append "root=/dev/sda1 console=ttyS0,115200n8 console=ttyS0"
```
Use SSH to login to the machine
```
ssh -p 2222 ankit@localhost
```

* Create a new file /etc/init/ttyS0.conf
```
# ttyS0 - getty
#
# This service maintains a getty on ttyS0 from the point the system is
# started until it is shut down again.
start on stopped rc or RUNLEVEL=[2345]
stop on runlevel [!2345]
respawn
exec /sbin/getty -L 115200 ttyS0 vt102

```

### Debuggin Kernel

There are several ways to start gdbserver so that gdb can attach remotely.

You can either start the VM using `-s` flag and then you can attach gdb remotely. Note: `-s` is equivalent to `-gdb tcp::1234`
```
sudo qemu-system-x86_64 -drive file=tmpPKS3JS.qcow2 --snapshot -net nic,model=virtio -net user -redir tcp:2222::22 -nographic -kernel ~/workspace/kernel/kbuild/arch/x86_64/boot/bzImage -boot c -append "root=/dev/sda1 console=ttyS0,115200n8 console=ttyS0" -s
```

To attach gdb remotely:
```
bash> gdb workspace/kernel/kbuild/vmlinux
gdb> target remote:1234
```
More Info: http://people.eecs.ku.edu/~kpoduval/UMOD/kvm_gdb.html

OR

#### Qemu monitor
Another way to start gdbserver is by using qemu-monitor console. User `ctrl + alt + shift + 2` to go to qemu console and run
```
(qemu) gdbserver
```

##### Other useful qemu monitor commands

```sh
(qemu) commit all # Commit your changes to image when running using -snapshot option

(qemu) system_powerdown # to shutdown vm cleanly

```
More on qemu monitor: http://en.wikibooks.org/wiki/QEMU/Monitor

Information on registers, commands:

http://www.cs.nyu.edu/~mwalfish/classes/ut/s12-cs372h/labs/labguide.html#make


#### Debugging process

Program used for debugging:

```c
#include<unistd.h>
#include<fcntl.h>
int main()
{
  int fd = open("/dev/urandom", O_RDONLY);
  char data[4096];
  read(fd, &data, 4096);
  close(fd);
  fd = open("/dev/null", O_WRONLY);
  write(fd, &data, 4096);
  close(fd);
}
```

* swapper is an idle process with id = 0. It runs when no other process is runnnin.


### Errors

Ignore these errors. These errors occurs irrespective of how you launch the kernel.

```
init: plymouth main process (80) killed by SEGV signal
init: plymouth-splash main process (312) terminated with status 2
mountall: Plymouth command failed
mountall: Plymouth command failed
mountall: Plymouth command failed
mountall: Plymouth command failed
mountall: Disconnected from Plymouth
init: plymouth-log main process (323) terminated with status 1
```

useful links:
http://www.linuxchix.org/content/courses/kernel_hacking/lesson8
