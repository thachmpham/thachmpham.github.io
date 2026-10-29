---
title: 'Systemd Unit Hang During Boot'
subtitle: '(Reproduce & Troubleshoot Series)'
---


# Overview
A systemd unit hang during boot, before the login prompt.


# Lab
- Setup lab: [Fedora 1 Node](/html/lab/fedora_1n.html)


# Reproduce
- Install httpd.
```sh
vm$ dnf install httpd
```

- Create a loop script.
```sh
vm$ vi /usr/bin/loop.sh
# -------------------- #
#!/usr/bin/bash

for i in {1..300}; do
        echo "httpd: loop $i"
        sleep 1
done

echo "httpd: ready"
systemd-notify --ready

echo "httpd: exit"
exit 1
# -------------------- #

vm$ chmod 777 /usr/bin/loop.sh
```

- Modify httpd service: Start from the loop.sh file, and before the login promt serial-getty@ttyS0.service.
```sh
vm$ systemctl cat httpd.service
# /usr/lib/systemd/system/httpd.service

vm$ vi /usr/lib/systemd/system/httpd.service
[Unit]
Before=serial-getty@ttyS0.service
Wants=serial-getty@ttyS0.service

[Service]
Type=notify
ExecStart=/usr/bin/loop.sh
ExecStop=pkill loop.sh

vm$ systemctl daemon-reload

vm$ systemctl list-dependencies httpd.service --before
httpd.service
  ├─serial-getty@ttyS0.service
  └─shutdown.target
...

vm$ systemctl list-dependencies serial-getty@ttyS0.service --after
serial-getty@ttyS0.service
  ├─dev-ttyS0.device
  ├─httpd.service
...
```

- Enable to start httpd.service on boot.
```sh
vm$ systemctl enable httpd
```

- Reboot.
```sh
vm$ reboot
# reboot hang for 5 minutes without login prompt
# because http.service stuck in loop.sh
#
# timeline:
#   http.service
#       => /usr/bin/loop.sh (5min)
#   serial-getty@ttyS0.service
#       => login prompt
```


# Troubleshoot
## Tip 1: Use Rescue Mode
- Use the kernel parameter 'systemd.unit=rescue.target' to boot to rescue mode.
```sh
                               GRUB version 2.12

 +----------------------------------------------------------------------------+
 |load_video                                                                  |
 |set gfxpayload=keep                                                         |
 |insmod gzio                                                                 |
 |linux ($root)/vmlinuz-6.19.10-300.fc44.x86_64 root=/dev/mapper/systemVG-LVR\|
 |oot ro  no_timer_check console=tty1 console=ttyS0,115200n8 systemd.unit=res\|
 |uce.target                                                                  |
 |initrd ($root)/initramfs-6.19.10-300.fc44.x86_64.img                        |
 +----------------------------------------------------------------------------+

      Minimum Emacs-like screen editing is supported. TAB lists
      completions. Press Ctrl-x or F10 to boot, Ctrl-c or F2 for
      a command-line or ESC to discard edits and return to the GRUB menu.
```

- Enable boot menu if it is not shown during boot.
```sh
vm$ vi /etc/default/grub
# -------------------- #
GRUB_TERMINAL_INPUT="serial"
GRUB_TERMINAL_OUTPUT="serial"
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=10
GRUB_CMDLINE_LINUX_DEFAULT="no_timer_check console=tty1 console=ttyS0,115200n8"
# -------------------- #

vm$ grub2-mkconfig -o /boot/grub2/grub.cfg
```


## Tip 2: Forward Journal Log to Console
- Use the kernel parameters 'systemd.journald.forward_to_console=1' to forward journal log to console.
```sh
                               GRUB version 2.12

 +----------------------------------------------------------------------------+
 |load_video                                                                  |
 |set gfxpayload=keep                                                         |
 |insmod gzio                                                                 |
 |linux ($root)/vmlinuz-6.19.10-300.fc44.x86_64 root=/dev/mapper/systemVG-LVR\|
 |oot ro  no_timer_check console=tty1 console=ttyS0,115200n8 systemd.journald\|
 |.forward_to_console=1                                                       |
 |initrd ($root)/initramfs-6.19.10-300.fc44.x86_64.img                        |
 +----------------------------------------------------------------------------+

      Minimum Emacs-like screen editing is supported. TAB lists
      completions. Press Ctrl-x or F10 to boot, Ctrl-c or F2 for
      a command-line or ESC to discard edits and return to the GRUB menu.
```

- The log of loop.sh will be shown in the console.
```sh
[   39.843013] loop.sh[1007]: httpd: loop 1
[   40.885196] loop.sh[1007]: httpd: loop 2
...
[   60.393537] loop.sh[1007]: httpd: loop 21
[   61.418861] loop.sh[1007]: httpd: loop 22
...
```


## Tip 3: Use Additional Console
- Modify the start_vm.sh script. Use parameter '-serial pty:/tmp/pty' to redirect VM console ttyS1 to file /tmp/pty on host.

```sh
cont$ vi /ws/start_vm.sh
# -------------------- #
#!/usr/bin/bash

disk_a="Fedora-Server-Guest-Generic-44-1.7.x86_64.qcow2"
disk_b="disk_b.qcow2"
disk_c="disk_c.qcow2"

qemu-system-x86_64 -name vm \
    -m 4096 \
    -smp cpus=2 \
    -nographic \
    -drive file=$disk_a \
    -drive file=$disk_b \
    -drive file=$disk_c \
    -netdev user,id=net0 -device virtio-net-pci,netdev=net0 \
    -netdev bridge,br=br1,id=net1 -device virtio-net-pci,netdev=net1,mac=50:54:00:01:00:01 \
    -netdev bridge,br=br2,id=net2 -device virtio-net-pci,netdev=net2,mac=50:54:00:02:00:01 \
    -netdev bridge,br=br3,id=net3 -device virtio-net-pci,netdev=net3,mac=50:54:00:03:00:01 \
    -serial mon:stdio \
    -serial pty:/tmp/pty
# -------------------- #
```

- Boot to rescue mode with parameters 'systemd.unit=rescue.target'.
```sh
                               GRUB version 2.12

 +----------------------------------------------------------------------------+
 |load_video                                                                  |
 |set gfxpayload=keep                                                         |
 |insmod gzio                                                                 |
 |linux ($root)/vmlinuz-6.19.10-300.fc44.x86_64 root=/dev/mapper/systemVG-LVR\|
 |oot ro  no_timer_check console=tty1 console=ttyS0,115200n8 console=ttyS1,11\|
 |5200n8 systemd.unit=rescue.target                                           |
 |initrd ($root)/initramfs-6.19.10-300.fc44.x86_64.img                        |
 +----------------------------------------------------------------------------+

      Minimum Emacs-like screen editing is supported. TAB lists
      completions. Press Ctrl-x or F10 to boot, Ctrl-c or F2 for
      a command-line or ESC to discard edits and return to the GRUB menu.
```


- Start ttyS0 and ttyS0 consoles.
```sh
vm$ systemctl start serial-getty@ttyS0.service
vm$ systemctl start serial-getty@ttyS1.service
```


- Find device on host that connect to VM console ttyS1.
```sh
vm$

# switch to qemu control: ctrl-a c

(qemu) info chardev
serial1: filename=pty:/dev/pts/2
parallel0: filename=null
serial0: filename=mux
serial0-base: filename=stdio
```

- Access to ttyS1.
```sh
cont$ screen /dev/pts/2 115200n8
# or: screen /tmp/pty 115200n8

# to exit screen: ctrl-a k
```

- On ttyS0, start isolate default.target.
```sh
vm$ systemctl isolate default.target
```

- On ttyS1, watch systemd jobs.
```sh
vm$ watch systemctl list-jobs
Every 2.0s: systemctl list-jobs

JOB  UNIT             TYPE  STATE
830 httpd.service     start running
645 multi-user.target start waiting
```

- From the job list on ttyS1, we can identify that the boot stuck due to httpd.service whose state remains in running for a long time.


# References
- [Diagnosing Boot Problems](https://systemd.io/DEBUGGING/)
- [Gettys on Serial Consoles](https://0pointer.de/blog/projects/serial-console.html)
- [Learning to love systemd](https://opensource.com/article/20/4/systemd)
- [QEMU Create Second Serial Port](https://stackoverflow.com/questions/52801787/qemu-doesnt-create-a-second-serial-port-ubuntu-x86-64-guest-and-host)
