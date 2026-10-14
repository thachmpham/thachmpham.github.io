---
title: 'Slow Disk'
subtitle: '(Fault Injection Series)'
---


# Overview

| Term/Tool | Usage | 
| :--- | :--- |
| Device Mapper (DM) | Map software devices to physical device. |
| Loop Device | Allow to mount a regular file as a disk. |
| dmsetup | Manage DM devices. |
| losetup | Manage loop device. |
| dm-linear | Merge multiple devices into one. Split one device into multiple devices. |
| dm-error  | Return I/O errors for any read, write to an erea. |

Agenda:

- Create a regular file.
- Create a loop device associated with the file.
- Create a dm-linear device on top of the loop device.
- Inject dm-error an erea of the dm-linear device.
- Read, write to the dm-error erea.


# References
- https://kb.linbit.com/generic-linux-tips/dmsetup-creating-virtual-blkdev-to-generate-io-errors/
- https://kb.linbit.com/generic-linux-tips/how-to-simulate-disk-io-error/
- https://blogs.oracle.com/linux/emulating-disk-latency-with-dm-delay
- https://wiki.gentoo.org/wiki/Device-mapper