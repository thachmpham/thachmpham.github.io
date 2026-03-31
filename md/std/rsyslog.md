---
title: "rsyslog"
---

# Build & Run
- Install dependencies.
```sh
$ apt install -y build-essential pkg-config libestr-dev libfastjson-dev zlib1g-dev \
uuid-dev libgcrypt20-dev liblogging-stdlog-dev libhiredis-dev \
uuid-dev liblogging-stdlog-dev flex bison libcurl4-openssl-dev
```

- Download source code.
```sh
$ wget http://www.rsyslog.com/download/files/download/rsyslog/rsyslog-8.2110.0.tar.gz
$ tar xf rsyslog-8.2110.0.tar.gz -C rsyslog
```

- Build.
```sh
$ cd rsyslog
$ ./configure --enable-debug
$ make
```

- Run without install.
```sh
$ rsyslog/tools/rsyslogd
```

- Or install and run.
```sh
$ make install
$ /usr/local/sbin/rsyslogd
```


# Reverse Engineering
## Memory to C Struct
### struct lstn_t

:::::::::::::: {.columns}
::: {.column width=50%}

- Check the memory with GDB.

```sh
(gdb) break readSocket
Breakpoint 1 at 0x7f11fdd37bf0: file imuxsock.c, line 1051.

(gdb) c
Continuing.
Thread 2 "in:imuxsock" hit Breakpoint 1, readSocket (pLstn=0x556bca210480) at imuxsock.c:1051

(gdb) ptype lstn_t
type = struct lstn_s {
    uchar *sockName;
    prop_t *hostName;
    int fd;
    int flags;
    int flowCtl;
    unsigned int ratelimitInterval;
    unsigned int ratelimitBurst;
    ratelimit_t *dflt_ratelimiter;
    intTiny ratelimitSev;
    struct hashtable *ht;
    sbool bParseHost;
    sbool bCreatePath;
    sbool bUseCreds;
    sbool bAnnotate;
    sbool bParseTrusted;
    sbool bWritePid;
    sbool bDiscardOwnMsgs;
    sbool bUseSysTimeStamp;
    sbool bUnlink;
    sbool bUseSpecialParser;
    ruleset_t *pRuleset;
}

(gdb) p sizeof(lstn_t)
$3 = 88

(gdb) p pLstn
$1 = (lstn_t *) 0x556bca210480

(gdb) p *pLstn
$2 = {sockName = 0x7f11fdd3b394 "/dev/log", hostName = 0x0, fd = 3, flags = 4, flowCtl = 0, ratelimitInterval = 0, ratelimitBurst = 200, dflt_ratelimiter = 0x556bca2232e0, ratelimitSev = 1 '\001', ht = 0x0, bParseHost = -1 '\377', bCreatePath = 0 '\000', bUseCreds = 1 '\001', bAnnotate = 0 '\000', bParseTrusted = 0 '\000', bWritePid = 0 '\000', bDiscardOwnMsgs = 0 '\000', bUseSysTimeStamp = 1 '\001', bUnlink = 1 '\001', bUseSpecialParser = 1 '\001', pRuleset = 0x0}

(gdb) x/88bx 0x556bca210480
0x556bca210480: 0x94    0xb3    0xd3    0xfd    0x11    0x7f    0x00    0x00
0x556bca210488: 0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x556bca210490: 0x03    0x00    0x00    0x00    0x04    0x00    0x00    0x00
0x556bca210498: 0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x556bca2104a0: 0xc8    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x556bca2104a8: 0xe0    0x32    0x22    0xca    0x6b    0x55    0x00    0x00
0x556bca2104b0: 0x01    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x556bca2104b8: 0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x556bca2104c0: 0xff    0x00    0x01    0x00    0x00    0x00    0x00    0x01
0x556bca2104c8: 0x01    0x01    0x00    0x00    0x00    0x00    0x00    0x00
0x556bca2104d0: 0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00

(gdb) dump memory mem.hex 0x556bca210480 0x556bca210480+88
```

- Decode memory manually.

```sh
>>> xxd -c8 mem.hex
00000000: 94b3 d3fd 117f 0000  ........
00000008: 0000 0000 0000 0000  ........
00000010: 0300 0000 0400 0000  ........
00000018: 0000 0000 0000 0000  ........
00000020: c800 0000 0000 0000  ........
00000028: e032 22ca 6b55 0000  .2".kU..
00000030: 0100 0000 0000 0000  ........
00000038: 0000 0000 0000 0000  ........
00000040: ff00 0100 0000 0001  ........
00000048: 0101 0000 0000 0000  ........
00000050: 0000 0000 0000 0000  ........
```

```go
94b3 d3fd 117f 0000  sockName
0000 0000 0000 0000  hostName
0300 0000 0400 0000  fd (4B), flag (4B)
0000 0000 0000 0000  flowCtl, ratelimitInterval
c800 0000 0000 0000  ratelimitBurst (4B), padding (4B)
e032 22ca 6b55 0000  dflt_ratelimiter
0100 0000 0000 0000  ratelimitSev
0000 0000 0000 0000  ht
ff00 0100 0000 0001  b...
0101 0000 0000 0000  b...
0000 0000 0000 0000  pRuleSet
```

:::
::: {.column width=50%}

- Decode with python struct.

```c
struct lstn_s {
    uchar *sockName;    // P
    prop_t *hostName;   // P
    int fd;             // i
    int flags;          // i
    int flowCtl;        // i
    unsigned int ratelimitInterval; // I
    unsigned int ratelimitBurst;    // I
    ratelimit_t *dflt_ratelimiter;  // P
    intTiny ratelimitSev;           // c
    struct hashtable *ht;           // P
    sbool bParseHost;       // c
    sbool bCreatePath;      // c
    sbool bUseCreds;        // c
    sbool bAnnotate;        // c
    sbool bParseTrusted;    // c
    sbool bWritePid;        // c
    sbool bDiscardOwnMsgs;  // c
    sbool bUseSysTimeStamp; // c
    sbool bUnlink;          // c
    sbool bUseSpecialParser;// c
    ruleset_t *pRuleset;    // P
}

// fmt = PPiiiIIPcPccccccccccP
```

```python
import struct
from collections import namedtuple

raw = open('mem.hex', 'rb').read()

fmt = 'PPiiiIIPcPccccccccccP'
print(struct.calcsize(fmt))     # 88

data = struct.unpack(fmt, raw)

lstn_s = namedtuple('lstn_s',
'sockName hostName fd flags flowCtl \
ratelimitInterval ratelimitBurst dflt_ratelimiter ratelimitSev ht \
bParseHost bCreatePath bUseCreds bAnnotate bParseTrusted \
bWritePid bDiscardOwnMsgs bUseSysTimeStamp bUnlink bUseSpecialParser \
pRuleset')

record = lstn_s._make(data)

for k, v in record._asdict().items():
    print(f"{k:20} {v.hex() if isinstance(v,(bytes,bytearray)) else f'{v:#x}'}")
# sockName             0x7f11fdd3b394
# hostName             0x0
# fd                   0x3
# flags                0x4
# flowCtl              0x0
# ratelimitInterval    0x0
# ratelimitBurst       0xc8
# dflt_ratelimiter     0x556bca2232e0
# ratelimitSev         01
# ht                   0x0
# bParseHost           ff
# bCreatePath          00
# bUseCreds            01
# bAnnotate            00
# bParseTrusted        00
# bWritePid            00
# bDiscardOwnMsgs      00
# bUseSysTimeStamp     01
# bUnlink              01
# bUseSpecialParser    01
# pRuleset             0x0
```

:::
::::::::::::::


## IPC
### logger to rsyslogd
Start rsyslogd without debug log.

- File /etc/rsyslog.conf
```sh
$DebugLevel 0
```

- Start rsyslogd.
```sh
$ rsyslog/tools/rsyslogd
```

- strace logger.
```sh
$ strace -e trace=network --write=all --read=all -yy -f logger hello

socket(AF_UNIX, SOCK_DGRAM, 0)          = 3<UNIX:[30664]>
connect(3<UNIX:[30664]>, {sa_family=AF_UNIX, sun_path="/dev/log"}, 110) = 0
sendmsg(3<UNIX:[30664->30189]>, {msg_name=NULL, msg_namelen=0, msg_iov=[{iov_base="<13>Mar 31 13:59:51 root: ", iov_len=26}, {iov_base="hello", iov_len=5}], msg_iovlen=2, msg_controllen=0, msg_flags=0}, MSG_NOSIGNAL) = 31
 * 26 bytes in buffer 0
 | 00000  3c 31 33 3e 4d 61 72 20  33 31 20 31 33 3a 35 39  <13>Mar 31 13:59 |
 | 00010  3a 35 31 20 72 6f 6f 74  3a 20                    :51 root:        |
 * 5 bytes in buffer 1
 | 00000  68 65 6c 6c 6f                                    hello            |
+++ exited with 0 +++
```

- strace rsyslogd.
```sh
$ strace -yy -f -p `pidof rsyslogd`

[pid  6925] recvmsg(3<UNIX:[48529,"/dev/log"]>, {msg_name=NULL, msg_namelen=0, msg_iov=[{iov_base="<13>Mar 31 14:43:57 root: hello", iov_len=8096}], msg_iovlen=1, msg_control=[{cmsg_len=32, cmsg_level=SOL_SOCKET, cmsg_type=SO_TIMESTAMP_OLD, cmsg_data={tv_sec=1774968237, tv_usec=352869}}, {cmsg_len=28, cmsg_level=SOL_SOCKET, cmsg_type=SCM_CREDENTIALS, cmsg_data={pid=7229, uid=0, gid=0}}], msg_controllen=64, msg_flags=0}, MSG_DONTWAIT) = 31
[pid  6936] write(2</var/log/syslog>, "Mar 31 14:43:57 pc root: hello\n", 31) = 31
```

- Debug rsyslogd with gdb.
```sh
$ gdb -p `pidof rsyslogd`

(gdb) catch syscall recvmsg
Catchpoint 1 (syscall 'recvmsg' [47])

(gdb) catch syscall write
Catchpoint 2 (syscall 'write' [1])

(gdb) c
Continuing.
```

- rsyslogd handles incomming messages.
```sh
[Switching to Thread 0x7fa065194640 (LWP 6925)]
Thread 2 "in:imuxsock" hit Catchpoint 1 (call to syscall recvmsg), __recvmsg_syscall (flags=64, msg=0x7fa0651907e0, fd=3) at ../sysdeps/unix/sysv/linux/recvmsg.c:27

(gdb) info threads
  Id   Target Id                                        Frame 
  1    Thread 0x7fa0652c4780 (LWP 6924) "rsyslogd"      pselect64_syscall (sigmask=0x7ffe614c8e50, timeout=<optimized out>, exceptfds=0x0, writefds=0x0, readfds=0x0, nfds=0) at ../sysdeps/unix/sysv/linux/pselect.c:34
* 2    Thread 0x7fa065194640 (LWP 6925) "in:imuxsock"   __recvmsg_syscall (flags=64, msg=0x7fa0651907e0, fd=3) at ../sysdeps/unix/sysv/linux/recvmsg.c:27
  3    Thread 0x7fa064d93640 (LWP 6926) "in:imklog"     __GI___libc_read (nbytes=8096, buf=0x7fa064d72ce0, fd=5) at ../sysdeps/unix/sysv/linux/read.c:26
  4    Thread 0x7fa064992640 (LWP 6936) "rs:main Q:Reg" __futex_abstimed_wait_common64 (private=0, cancel=true, abstime=0x0, op=393, expected=0, futex_word=0x5619e6c60a0c) at ./nptl/futex-internal.c:57

(gdb) bt
#0  __recvmsg_syscall (flags=64, msg=0x7fa0651907e0, fd=3) at ../sysdeps/unix/sysv/linux/recvmsg.c:27
#1  __libc_recvmsg (fd=3, msg=msg@entry=0x7fa0651907e0, flags=flags@entry=64) at ../sysdeps/unix/sysv/linux/recvmsg.c:41
#2  0x00007fa0652bcce1 in readSocket (pLstn=0x5619e6c39480) at imuxsock.c:1105
#3  0x00007fa0652be013 in runInput (pThrd=<optimized out>) at imuxsock.c:1548
#4  0x00005619c2df059a in thrdStarter (arg=0x5619e6c5a890) at ../threads.c:243
#5  0x00007fa06535dac3 in start_thread (arg=<optimized out>) at ./nptl/pthread_create.c:442
#6  0x00007fa0653ef8d0 in clone3 () at ../sysdeps/unix/sysv/linux/x86_64/clone3.S:81

(gdb) c
Continuing.
```

- rsyslogd write messages to log file.
```sh
[Switching to Thread 0x7fa064992640 (LWP 6936)]
Thread 4 "rs:main Q:Reg" hit Catchpoint 2 (call to syscall write), __GI___libc_write (nbytes=31, buf=0x7fa05c001200, fd=2) at ../sysdeps/unix/sysv/linux/write.c:26

(gdb) info threads
  Id   Target Id                                        Frame 
  1    Thread 0x7fa0652c4780 (LWP 6924) "rsyslogd"      pselect64_syscall (sigmask=0x7ffe614c8e50, timeout=<optimized out>, exceptfds=0x0, writefds=0x0, readfds=0x0, nfds=0) at ../sysdeps/unix/sysv/linux/pselect.c:34
  2    Thread 0x7fa065194640 (LWP 6925) "in:imuxsock"   0x00007fa0653e1c4f in __GI___poll (fds=fds@entry=0x7fa060000b90, nfds=1, timeout=timeout@entry=-1) at ../sysdeps/unix/sysv/linux/poll.c:29
  3    Thread 0x7fa064d93640 (LWP 6926) "in:imklog"     __GI___libc_read (nbytes=8096, buf=0x7fa064d72ce0, fd=5) at ../sysdeps/unix/sysv/linux/read.c:26
* 4    Thread 0x7fa064992640 (LWP 6936) "rs:main Q:Reg" __GI___libc_write (nbytes=31, buf=0x7fa05c001200, fd=2) at ../sysdeps/unix/sysv/linux/write.c:26

(gdb) bt
#0  __GI___libc_write (nbytes=31, buf=0x7fa05c001200, fd=2) at ../sysdeps/unix/sysv/linux/write.c:26
#1  __GI___libc_write (fd=2, buf=buf@entry=0x7fa05c001200, nbytes=nbytes@entry=31) at ../sysdeps/unix/sysv/linux/write.c:24
#2  0x00005619c2dd1de1 in doWriteCall (pLenBuf=<synthetic pointer>, pBuf=<optimized out>, pThis=0x7fa05c000f10) at stream.c:1464
#3  strmPhysWrite (pThis=pThis@entry=0x7fa05c000f10, pBuf=<optimized out>, lenBuf=<optimized out>) at stream.c:1770
#4  0x00005619c2dd2491 in doWriteInternal (pThis=pThis@entry=0x7fa05c000f10, pBuf=<optimized out>, lenBuf=<optimized out>, bFlush=0, bFlush@entry=1) at stream.c:1531
#5  0x00005619c2dd28e4 in strmSchedWrite (bFlushZip=1, lenBuf=<optimized out>, pBuf=<optimized out>, pThis=0x7fa05c000f10) at stream.c:1604
#6  0x00005619c2dd2f9a in strmFlush (pThis=0x7fa05c000f10) at stream.c:1933
#7  0x00005619c2d95658 in commitTransaction (pWrkrData=<optimized out>, pParams=0x7fa05c000b90, nParams=1) at omfile.c:1078
#8  0x00005619c2ded0e2 in actionCallCommitTransaction (nparams=<optimized out>, iparams=<optimized out>, pWti=<optimized out>, pThis=<optimized out>) at ../action.c:1283
#9  doTransaction (nparams=1, iparams=0x7fa05c000b90, pWti=0x5619e6c60970, pThis=0x5619e6c52f10) at ../action.c:1327
#10 actionTryCommit (nparams=1, iparams=0x7fa05c000b90, pWti=0x5619e6c60970, pThis=0x5619e6c52f10) at ../action.c:1369
#11 actionTryCommit (pThis=0x5619e6c52f10, pWti=0x5619e6c60970, iparams=0x7fa05c000b90, nparams=1) at ../action.c:1361
#12 0x00005619c2ded69f in actionCommit (pThis=pThis@entry=0x5619e6c52f10, pWti=pWti@entry=0x5619e6c60970) at ../action.c:1543
#13 0x00005619c2deefcc in actionCommitAllDirect (pWti=pWti@entry=0x5619e6c60970) at ../action.c:1632
#14 0x00005619c2de6053 in processBatch (pBatch=0x5619e6c609a8, pWti=0x5619e6c60970) at ruleset.c:675
#15 0x00005619c2d8afa4 in msgConsumer (notNeeded=<optimized out>, pBatch=0x5619e6c609a8, pWti=0x5619e6c60970) at rsyslogd.c:694
#16 0x00005619c2ddfa5f in ConsumerReg (pThis=0x5619e6c58980, pWti=0x5619e6c60970) at queue.c:2164
#17 0x00005619c2dda859 in wtiWorker (pThis=pThis@entry=0x5619e6c60970) at wti.c:428
#18 0x00005619c2dd7b85 in wtpWorker (arg=0x5619e6c60970) at wtp.c:435
#19 0x00007fa06535dac3 in start_thread (arg=<optimized out>) at ./nptl/pthread_create.c:442
#20 0x00007fa0653ef8d0 in clone3 () at ../sysdeps/unix/sysv/linux/x86_64/clone3.S:81
```

- Illustrate.
<pre class="mermaid">
sequenceDiagram
    participant logger
    participant pipe@{"type": "queue"} as /dev/log
    
    box DarkBlue rsyslogd
    participant uxsock as in:imuxsock
    participant rsmain as rs:main
    end

    participant logfile as /var/log/syslog

    logger ->> pipe: hello
    pipe ->> uxsock: hello
    uxsock ->> rsmain: hello
    rsmain ->> logfile: hello
</pre>