---
title: 'NFS Server Not Responding'
subtitle: '(Reproduce Series)'
---


# Overview
According to [NFS manual](https://linux.die.net/man/5/nfs), the NFS client can specify the mount options with the below parameters.

```sh
nfs_client$ mount -t nfs -o recovery_method,timeo=n,retrans=n server:path path
```

- recovery_method: How the client recovers when an NFS request times out.
    - hard: Retry indefinitely. Default.
    - soft: Retry a limited number of times.
- timeo: Time to wait before retry. Unit: decisecond (0.1s). Default: 600 deciseconds.
- retrans: Number of retries before apply recovery method. Default: 3.

"NFS server not responding" occurs when the NFS client cannot complete a RPC function within the timeout. Common reasons:

- Firewall block: Firewall settings block the NFS ports. Requests cannot reach the server.
- Network errors: Packet drop, high latency. Messages cannot reach NFS peers in time.
- Disk IO bottleneck: The server experiences massive write loads, cannot read or write data fast enough to reply in time.
- Resource exhausted: The server handles too many requests or suffers from CPU/RAM starvation, unable to accept new requests.

To quickly reproduce "NFS server not responding", we use small timeo, retrans values and simulate the above conditions.


# Prepare Lab
- Setup lab: [Network File System (NFS)](/html/lab/nfs.html).
- VM1 acts as NFS server.
- VM2 acts as NFS client.


# Reproduce
## Firewall Blocks NFS


## NFS Fails due to Network Errors


## Disk IO Bottleneck


## Resource Exhausted



# References