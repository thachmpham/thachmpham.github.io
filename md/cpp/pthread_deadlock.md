---
title: Analyze PThread Deadlock
---

# Syscall Futex
Syscall futex provides:

- FUTEX_WAIT: The method for a thread to sleep await a value at a given address.
- FUTEX_WAKE: The method to wake up thread through the given address.

```c
int futex(int *uaddr, int op, int val, const struct timespec *timeout, int *uaddr2, int val3)
```

- uaddr: target address.
- op: FUTEX_WAIT, FUTEX_WAKE.
- val: behavior depends on parameter op.

Syscall futex is used to implement pthread mutex.

## Futex Wait
```c
futex(uaddr, FUTEX_WAIT, val, timeout, ...)
```

- If *uaddr == val, futex lets the thread sleeps awaiting FUTEX_WAKE on uaddr.
- Otherwise, no waits, futex returns EAGAIN immediately, the thread continues.

**Unit Test**.

:::::::::::::: {.columns}
::: {.column}

1. futex_wait.c
```c
#include <linux/futex.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <pthread.h>
#include <stdlib.h>

int* pInt = NULL;

void* do_wait(void*)
{
    // wait only if *pInt == 5
    syscall(SYS_futex, pInt, FUTEX_WAIT_PRIVATE, 5, NULL, NULL, 0);
}

int main(int argc, char** argv)
{
    pthread_t t1;

    pInt = malloc(sizeof(int));
    *pInt = atoi(argv[1]);

    pthread_create(&t1, NULL, do_wait, NULL);

    sleep(10);
}
```

:::
::: {.column}

2. Build.
```sh
$ gcc -g -o futex_wait futex_wait.c -pthread
```

3. Case `*uaddr == val`: futex lets the thread sleep waits.
```sh
$ strace -tt -e futex -f ./futex_wait 5
strace: Process 7143 attached
[pid  7143] 11:21:05.931247 futex(0x3bc2e2a0, FUTEX_WAIT_PRIVATE, 5, NULL) = ?
[pid  7143] 11:21:15.931846 +++ exited with 0 +++
```
- 11:21:05, futex wait, *uaddr == val, so sleep await.
- 11:21:15, process exits, thread exits.

4. Case `*uaddr != val`: futex returns EAGAIN, no waits.
```sh
$ strace -tt -e futex -f ./futex_wait 4
strace: Process 7068 attached
[pid  7068] 11:18:39.094043 futex(0x289302a0, FUTEX_WAIT_PRIVATE, 5, NULL) = -1 EAGAIN (Resource temporarily unavailable)
[pid  7068] 11:18:39.096230 +++ exited with 0 +++

```
- 11:18:39, futex wait, *uaddr == val, so don't wait, futex returns EAGAIN immediately. Thread continues.
- 11:18:39, thread exits after complete entry function.

:::
::::::::::::::

## Futex Wake
```c
futex(uaddr, FUTEX_WAKE, val, _)
```
- Wakes at most val processes waiting on uaddr.

**Unit Test**

:::::::::::::: {.columns}
::: {.column}

- 1. futex_wake.c
```c
#include <linux/futex.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <pthread.h>
#include <stdlib.h>

int* pInt = NULL;

void* do_wait(void*)
{
    // wait only if *pInt == 5
    syscall(SYS_futex, pInt, FUTEX_WAIT_PRIVATE, 5, NULL, NULL, 0);
}

void* do_wake(void*)
{
    sleep(10);
    
    // wake at most 1 waiter thread
    syscall(SYS_futex, pInt, FUTEX_WAKE_PRIVATE, 1, NULL, NULL, 0);
}

int main(int argc, char** argv)
{
    pthread_t t1, t2;

    pInt = malloc(sizeof(int));
    *pInt = 5;

    pthread_create(&t1, NULL, do_wait, NULL);
    pthread_create(&t2, NULL, do_wake, NULL);

    sleep(20);
}
```

:::
::: {.column}

2. Build.
```sh
$ gcc -g -o futex_wake futex_wake.c -pthread
```

3. Run.
```sh
>>> strace -tt -e futex -f ./futex_wake
strace: Process 7313 attached
[pid  7313] 11:26:02.068129 futex(0x2b9752a0, FUTEX_WAIT_PRIVATE, 5, NULL <unfinished ...>
strace: Process 7314 attached
[pid  7314] 11:26:12.074906 futex(0x2b9752a0, FUTEX_WAKE_PRIVATE, 1) = 1
[pid  7313] 11:26:12.075414 <... futex resumed>) = 0
[pid  7313] 11:26:12.077728 +++ exited with 0 +++
[pid  7314] 11:26:12.078209 +++ exited with 0 +++
```

- 11:26:02, thread 7313 sleeps await on 0x2b9752a0.
- 11:26:12, thread 7314 send wake-up notification to 0x2b9752a0.
- 11:26:12, thread 7313 wakes up and resumes.

:::
::::::::::::::


# Deadlock
## Self-Relock Deadlock
Scenario

- Step 1: Thread 1 locks mutex A.
- Step 2: Thread 1 relocks mutex A.

At step 2, thread 1 sleeps await mutex A. But the mutex is owned by thread 1, other threads cannot unlock the mutex. The mutex becomes locked forever, so thread 1 sleeps forever.

**Reproduce & Troubleshoot**

:::::::::::::: {.columns}
::: {.column}

1. deadlock_self_relock.c
```c
#include <pthread.h>

int main(int argc, char* argv)
{
    pthread_mutex_t mutex_a;
    pthread_mutex_init(&mutex_a, NULL);
    pthread_mutex_lock(&mutex_a);
    pthread_mutex_lock(&mutex_a);
}
```

```sh
$ gcc -g -o deadlock_self_relock deadlock_self_relock.c -pthread

$ ./deadlock_self_relock
# process hangs
```

:::
::: {.column}

2. Detect self deadlock.
- Trace syscall futex.
```sh
$ strace -e futex -f -p `pidof deadlock_self_relock`
strace: Process 9526 attached
futex(0xffffe8f67ea0, FUTEX_WAIT_PRIVATE, 2, NULL

# thread 9526 waits for mutex 0xffffe8f67ea0.
```

- Print mutex content.
```sh
$ gdb -batch -p `pidof deadlock_self_relock` -ex 'p *(pthread_mutex_t*)0xffffe8f67ea0'
$1 = {__data = {__lock = 2, __count = 0, __owner = 9526, __nusers = 1, __kind = 0, __spins = 0, __list = {__prev = 0x0, __next = 0x0}}, __size = "\002\000\000\000\000\000\000\0006%\000\000\001", '\000' <repeats 34 times>, __align = 2}

# mutex 0xffffe8f67ea0 is owned by thread 9526.
```

- Thead 9526 waits for mutex 0xffffe8f67ea0 which is owned by the thread itself.

3. Workaround: Force unlock.
```sh
# find ID of thread 9526
$ gdb -batch -p `pidof deadlock_self_relock` -ex 'thread find 9526'
Thread 1 has target id 'Thread 0xffffa08c9e40 (LWP 9526)'

# thread LWP 9526 has ID 1
```

```sh
# unlock the mutex from the owner thread.

$ gdb -batch -p `pidof deadlock_self_relock` -ex 'thread apply 1 call (int) pthread_mutex_unlock((pthread_mutex_t*)0xffffe8f67ea0)'
[Inferior 1 (process 9526) detached]

# After unlock, the thread wakes up, the program continues.
```

:::
::::::::::::::

Solution: Self-Relock deadlock can be solved by PTHREAD_MUTEX_RECURSIVE.


## ABBA Deadlock
Scenario
```sh
Step 1: Thread 1 locks mutex A. |
                                |   Step 2: Thread 2 locks mutex B.
Step 3: Thread 1 locks mutex B. |
                                |   Step 4: Thread 2 locks mutex A. 
```

- At step 3, thread 1 sleeps await mutex B which is owned by thread 2.
- At step 4, thread 2 sleeps await mutex A which is owned by thread 1.
- Both threads sleeps, no longer execute instructions, cannot reach the code to unlock mutexes. Program hangs.

**Reproduce & Troubleshoot**

:::::::::::::: {.columns}
::: {.column}

1. deadlock_abba.c
```c
#include <pthread.h>
#include <stdio.h>

pthread_mutex_t mutex_a, mutex_b;

void* do_lock_ab(void*)
{
    pthread_mutex_lock(&mutex_a);
    pthread_mutex_lock(&mutex_b);

    pthread_mutex_unlock(&mutex_b);
    pthread_mutex_unlock(&mutex_a);
}

void* do_lock_ba(void*)
{
    pthread_mutex_lock(&mutex_b);
    pthread_mutex_lock(&mutex_a);

    pthread_mutex_unlock(&mutex_a);
    pthread_mutex_unlock(&mutex_b);
}

int main(int argc, char* argv)
{
    pthread_t t1, t2;
    pthread_mutex_init(&mutex_a, NULL);
    pthread_mutex_init(&mutex_b, NULL);

    pthread_create(&t1, NULL, do_lock_ab, NULL);
    pthread_create(&t2, NULL, do_lock_ba, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    printf("done\n");
}
```

```sh
$ gcc -g -o deadlock_abba deadlock_abba.c -pthread
```

2. Run
- Deadlock ABBA occurs randomly. So, repeatedly run the program until deadlock occurs. Terminal will hang on deadlock.

```sh
$ while true; do ./deadlock_abba; done
# process hangs
```

:::
::: {.column}

3. Detech deadlock ABBA.
- Trace syscall futex.
```sh
$ strace -e futex -f -p `pidof deadlock_abba`
strace: Process 5263 attached with 3 threads
[pid  5263] futex(0xffffa256f270, FUTEX_WAIT_BITSET|FUTEX_CLOCK_REALTIME, 5264, NULL, FUTEX_BITSET_MATCH_ANY <unfinished ...>
[pid  5264] futex(0x420088, FUTEX_WAIT_PRIVATE, 2, NULL <unfinished ...>
[pid  5265] futex(0x420058, FUTEX_WAIT_PRIVATE, 2, NULL

# thread 5264 waits for mutex 0x420088.
# thread 5265 waits for mutex 0x420058.
```

- Print mutex contents.
```sh
$ gdb -batch -p `pidof deadlock_abba` -ex 'p *(pthread_mutex_t*)0x420088'
$1 = {__data = {__lock = 2, __count = 0, __owner = 5265, __nusers = 1, __kind = 0, __spins = 0, __list = {__prev = 0x0, __next = 0x0}}, __size = "\002\000\000\000\000\000\000\000\221\024\000\000\001", '\000' <repeats 34 times>, __align = 2}

$ gdb -batch -p `pidof deadlock_abba` -ex 'p *(pthread_mutex_t*)0x420058'
$1 = {__data = {__lock = 2, __count = 0, __owner = 5264, __nusers = 1, __kind = 0, __spins = 0, __list = {__prev = 0x0, __next = 0x0}}, __size = "\002\000\000\000\000\000\000\000\220\024\000\000\001", '\000' <repeats 34 times>, __align = 2}

# mutex 0x420088 is owned by thread 5265.
# mutex 0x420058 is owned by thread 5264.
```

- Thread 5264 waits for mutex 0x420088 which is owned by 5265. Thread 5265 waits for mutex 0x420058 which is owned by 5264. The two threads waits for mutexes owned by each other.

4. Workaround: Force unlock a mutex.
```sh
# Unlock the mutex on thread 5264.

$ gdb -batch -p `pidof deadlock_abba` -ex 'thread find 5264'
Thread 2 has target id 'Thread _ (LWP 5264)'

$ gdb -batch -p `pidof deadlock_abba` -ex 'thread apply 2 call (int) pthread_mutex_unlock((pthread_mutex_t*)0x420088)'

# after unlock, deadlock solved, program can continue.
```

:::
::::::::::::::