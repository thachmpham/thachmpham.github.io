---
title:  'Systemd'
---


# 1. Configuration
- Paths:
   - `/etc/systemd`
   - `/usr/lib/systemd`

- Manual.
```sh
  
$ man service.unit

$ apropos systemd
  
```


# 2. systemctl
- List units.
```sh
  
$ systemctl list-units

$ systemctl list-unit-files
  
```

- Inspect an unit.
```sh
  
$ systemctl status rsyslog

$ systemctl show rsyslog
  
```

- Start, stop an unit.
```sh
  
$ systemctl start rsyslog

$ systemctl stop rsyslog

$ systemctl restart rsyslog
  
```

- Enable, disable an unit.
```sh
  
$ systemctl enable rsyslog

$ systemctl show rsyslog
  
```


# 3. systemd-analyze
```sh
  
$ systemd-analyze critical-chain

$ systemd-analyze critical-chain rsyslog

$ systemd-analyze blame
  
```


# 4. journalctl
- Check boot.
```sh
  
# list previous boots and timestamps.
$ journalctl --list-boots

# show errors of previous boot.
$ journalctl --boot=-1 --priority=err
  
```

- Check an unit.
```sh
  
$ journalctl -u docker

# live log
$ journalctl -fu docker
  
```


# 5. systemd-run
- Generate a dynamic unit.
```sh
  
$ systemd-run --unit=myservice --description='My Service' ping google.com

$ systemctl status myservice

$ systemctl stop myservice
#     after stop, the unit disappears
  
```