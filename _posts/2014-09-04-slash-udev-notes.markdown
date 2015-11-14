---
layout: post
title: "/udev notes"
date: 2014-09-04 02:13:15 -0500
comments: true
categories: unix filesystem
---

### Problems with `/dev`

1. A static /dev is unwiedly big. It would be nice to show only /dev enteries for the devices that we acutally have running in the system.
2. We are running out of major and minor number for devices.

```sh
  [vagrant@vagrant-centos64 ~]$ ls -l /dev/sda
  brw-rw---- 1 root disk 8, 0 Aug 14 05:36 /dev/sda

  # shows permissions (brw-rw---- ), owner (root), group (disk), major device number (8),
    minor device number (0), date (Aug 14), hour (05:36) and device name
```

  When accessing a device file, the major device number selects which device driver is being called to perform read/write. Minor number is passed as a parameter and its usage/interpretations depends on the driver.<br/>

3. Users want a way to name devices in persistant fashion.
4. Userspace programs wants to know what devices are created or removed and what /dev entry is associated with them.

<!-- more -->
### `udev`

Presentation on udev: http://www.kroah.com/linux/talks/ols_2003_udev_talk/mgp00001.html

1. the /dev tree is populated only for devices that are currently present in the system.
2. udev doesn't care about major or minor number schemes.
3. It provides ability to name devices persistently.
4. udev emites D-BUS messages so that any userspace program can listen to what devices are created or removed. Userspace programs can also query the database to see what devices are present and what they are currently named.

### Dynamic `/dev`

The content of /dev directory are stored on a temporary file system and all files are render at every system startup. Static files that should always be present in /dev directory can be placed in /lib/udev/devices directory. All the contents of /lib/udev/devices directory are copied to /dev at startup.

> how does kernel knows which device driver to choose without major or minor device number.

The kernel bus drivers probe for devices. For every detected devices kernel creates an internal device structure and sends a uevent to udev deamon. Each devices identifies itself by specially-formatted ID, which tells what kind of device it is. It usually consists of vendor ID, product ID and other subsystem specific values.

`/etc/udev/rules.d/*.rules` contains rules for different devices and `udev` deamon parses and loads all of them at the startup.

More Info at:
http://doc.opensuse.org/products/draft/SLES/SLES-admin_sd_draft/cha.udev.html#sec.udev.devdir

<hr/>
