---
title: 'Slow Disk'
subtitle: '(Fault Injection Series)'
---


# Overview

| Term/Tool | Usage | 
| :--- | :--- |
| Device Mapper (DM) | Map software devices to physical device. |
| Loop Device | Allow users to mount a regular file as a disk. |
| dmsetup | Manage DM devices. |
| losetup | Manage loop device. |
| dm-linear | Split, merge devices |
| dm-error  | Return I/O errors for any read, write to an erea. |

Steps:

- Create a regular file.
- Create a loop device on top of the file.
- Create a dm-linear device on top of the loop device.
- Inject errors to an erea of the dm-linear device.
- Test read, write on the error erea.


# References
- https://kb.linbit.com/generic-linux-tips/dmsetup-creating-virtual-blkdev-to-generate-io-errors/
- https://kb.linbit.com/generic-linux-tips/how-to-simulate-disk-io-error/
- https://blogs.oracle.com/linux/emulating-disk-latency-with-dm-delay
- https://wiki.gentoo.org/wiki/Device-mapper