---
title: "Traffic Control (tc)"
---

## Network Delay

:::::::::::::: {.columns}
::: {.column width=50%}

1. Simulate network delay.
```sh
$ tc qdisc add dev enp0s3 root netem delay 300ms
# tc qdisc change dev enp0s3 root netem delay 300ms

$ tc qdisc show dev enp0s3
qdisc netem 8001: root refcnt 2 limit 1000 delay 300ms
```

2. Ping test.
```sh
$ ping -c1 google.com
64 bytes from nchkga-ap-in-f14.1e100.net (142.250.198.46): icmp_seq=1 ttl=116 time=403 ms
```

:::
::: {.column width=50%}

3. Stop network delay simulation.
```sh
$ tc qdisc del dev enp0s3 root


$ tc qdisc show dev enp0s3
qdisc fq_codel 0: root refcnt 2 limit 10240p flows 1024 quantum 1514 target 5ms interval 100ms memory_limit 32Mb ecn drop_batch 64
```

4. Ping test.
```sh
$ ping -c1 google.com
64 bytes from nchkga-ap-in-f14.1e100.net (142.250.198.46): icmp_seq=1 ttl=116 time=57.7 ms
```

:::
::::::::::::::


## Packet Loss

:::::::::::::: {.columns}
::: {.column width=50%}

1. Simulate packet loss.
```sh
$ tc qdisc add dev enp0s3 root netem loss 20%

$ tc qdisc show dev enp0s3
qdisc netem 8002: root refcnt 2 limit 1000 loss 20%
```

:::
::: {.column width=50%}

2. Ping test.
```sh
$ ping -c 100 google.com
10 packets transmitted, 8 received, 20% packet loss, time 9189ms
```

:::
::::::::::::::


:::::::::::::: {.columns}
::: {.column width=50%}

## Packet Corruption
```sh
$ tc qdisc change dev enp0s3 root netem corrupt 5%
```

:::
::: {.column width=50%}

## Packet Duplication
```sh
$ tc qdisc change dev eth0 root netem duplicate 1%
```

:::
::::::::::::::
