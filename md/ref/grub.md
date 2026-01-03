---
title: "GRUB"
---

:::::::::::::: {.columns}
::: {.column width=40%}

Introduction

GRUB is a bootloader responsible for loading vmlinuz and initrd.img into RAM at booting:

- vmlinuz is the linux kernel.
- initrd.img contains tools and drivers to mount root filesystem.

:::
::: {.column width=30%}

Configuration Files 

- `/etc/default/grub`
    - User settings.
- `/etc/grub.d`
    - Helper files.
- `/boot/grub2/grub.cfg`
    - Read by grub at boot time.
    - Generated from /etc/default/grub.

:::
::: {.column width=30%}

```go
       /etc/default/grub    
               │            
grub-mkconfig │ update-grub
               │            
               ▼            
       /boot/grub2/grub.cfg 
```

:::
::::::::::::::

:::::::::::::: {.columns}
::: {.column width=30%}

Update configuration.
```sh
$ vi /etc/default/grub
$ update-grub
```

:::
::: {.column width=70%}


:::
::::::::::::::
