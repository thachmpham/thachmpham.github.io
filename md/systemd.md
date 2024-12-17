---
title:  'Systemd'
---


# 1. Configuration
- Service unit files located at **/etc/systemd/**.

- Manual for unit files.
```sh
  
$ man service.unit
  
```

- More documents.
```sh
  
$ apropos systemd
  
```

# 2. systemctl
- List service units.
```sh
  
$ systemctl list-units

# list failed units
$ systemctl list-units --state=failed

$ systemctl list-unit-files
  
```

- Inspect an unit.
```sh
  
$ systemctl status rsyslog

$ systemctl show rsyslog
  
```

- Start, stop an unit.
```sh
  
$ systemctl status rsyslog

$ systemctl show rsyslog
  
```

- Enable, disable an unit.
```sh
  
# start unit on boot
$ systemctl enable rsyslog
Created symlink /etc/systemd/system/syslog.service → /usr/lib/systemd/system/rsyslog.service.
Created symlink /etc/systemd/system/multi-user.target.wants/rsyslog.service → /usr/lib/systemd/system/rsyslog.service.

# not start unit on boot
$ systemctl show rsyslog
Removed "/etc/systemd/system/multi-user.target.wants/rsyslog.service".
Removed "/etc/systemd/system/syslog.service".
  
```


# 3. systemd-analyze
- List critical units in the boot process.
```sh
  
$ systemd-analyze critical-chain
  
```

- Print critical chain of an unit.
```sh
  
$ systemd-analyze critical-chain rsyslog
  
```

- List all running units, ordered by the time they took to initialize.
```sh
  
$ systemd-analyze blame
  
```


# 4. journalctl
- List previous boots and timestamps.
```sh
  
$ journalctl --list-boots
  
```

- Print errors of previous boot.
```sh
  
$ journalctl --boot=-1 --priority=err
# journalctl -b -1 -p err
  
```

- Check log of a unit.
```sh
  
$ journalctl -u docker

# live log
$ journalctl -fu docker
  
```


# 5. systemd-run
- Generate a dynamic unit.
```sh
  
$ systemd-run --unit=myservice --description='My Service' ping google.com
Running as unit: myservice.service; invocation ID: 1699db4d1db1409abe26f92506ec4fe1

$ systemctl status myservice
myservice.service - My Service
     Loaded: loaded (/run/systemd/transient/myservice.service; transient)
  Transient: yes
     Active: active (running) since Tue 2024-12-17 08:43:18 +07; 25s ago
   Main PID: 23891 (ping)
      Tasks: 1 (limit: 4383)
     Memory: 464.0K (peak: 968.0K)
        CPU: 12ms
     CGroup: /system.slice/myservice.service
             └─23891 /usr/bin/ping google.com

Dec 17 08:43:33 pc ping[23891]: 64 bytes from sb-in-x66.1e100.net (2404:6800:4003:c01::66): icmp_seq=>
Dec 17 08:43:34 pc ping[23891]: 64 bytes from sb-in-x66.1e100.net (2404:6800:4003:c01::66): icmp_seq=>
Dec 17 08:43:35 pc ping[23891]: 64 bytes from sb-in-x66.1e100.net (2404:6800:4003:c01::66): icmp_seq=>

$ systemctl stop myservice

# after stop, the unit disappears
$ systemctl status myservice
Unit myservice.service could not be found.
  
```