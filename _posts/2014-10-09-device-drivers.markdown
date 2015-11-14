---
layout: post
title: "Device Drivers"
date: 2014-10-09 16:34:14 -0500
comments: true
categories: 
---

Device driver used to interact with device. Major number is used to identify device driver.


* Character Device: writes and reads character by character. Operates in blocking way (synchronous, user must wait for completion). More common.
* Block Device: write/read block by block (larfer).
