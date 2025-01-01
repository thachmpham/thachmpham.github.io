---
title: 'GDB'
subtitle: '**Debug A High Availability Program**'
---


# 1. Introduction
High-availability (HA) systems need uninterrupted service, often using multi-threaded programs where some threads handle real-time tasks or must continue to respond to external events. That makes debugging HA programs more complex.

GDB supports non-stop mode that lets you pause specific threads for debugging while other threads keep running.


# 2. Lab
## 2.1. OpenSAF AMF Program
In this lab, we will the debug the below OpenSAF AMF program.

- [github.com/thachmpham/samples/tree/main/opensaf/worker_thread](https://github.com/thachmpham/samples/tree/main/opensaf/worker_thread)

```c
  
int main(int argc, char ** argv)
{
    //...    
    pthread_create(&thread, NULL, threadFunction, NULL);

    saAmfSelectionObjectGet(amfHandler, &monitorFd);    
    struct pollfd fds[1];
    fds[0].fd = monitorFd;
    fds[0].events = POLLIN;
    
    while (poll(fds, 1, -1) != -1)
    {
        if (fds[0].revents & POLLIN)
        {            
            saAmfDispatch(amfHandler, SA_DISPATCH_ONE);
        }
    }
}

void* threadFunction(void* arg)
{
    trace();
    for (int i = 0; ; i++)
	{
        trace("continue... i=%d\n", i);
        sleep(5);
    }
}
  
```


We will debug the program in two phases:

- Debug the entry point of the program when the service unit is started (Section 2.2).
- Debug the program while it is running (Section 2.3).


## 2.2. Debug Entrypoint
### 2.2.1. Problems
When debugging entry point of a AMF application, we encounters the below problems:

- The program starts as daemon via the command configured by attribute `saAmfCtRelPathInstantiateCmd` of class `SaAmfCompType`.
- If the AMF operation cannot finished within `saAmfCtDefClcCliTimeout` of class `SaAmfCompType`, AMF will assume that the operation failed and stop it.
- If the component does not register within `saAmfCompInstantiateTimeout` of class `SaAmfComp`, AMF will automatically kill and restart the process.

```sh
  
$ amf-find comptype
safVersion=1,safCompType=demo

$ immlist safVersion=1,safCompType=demo
Name                                               Type         Value(s)
========================================================================
saAmfCtRelPathInstantiateCmd                       SA_STRING_T  control.sh
saAmfCtDefInstantiateCmdArgv                       SA_STRING_T  start
saAmfCtDefClcCliTimeout                            SA_TIME_T    10000000000
  
```


### 2.2.1. Steps
- Modify service startup script.
```sh

```

- Extend saAmfCtDefClcCliTimeout enough to debug.
```sh
  
$ immcfg -a saAmfCtDefClcCliTimeout=60000000000 safVersion=1,safCompType=demo

$ immcfg -a saAmfCompInstantiateTimeout=60000000000 safComp=demo,safSu=SC-1,safSg=demo,safApp=demo
  
```



# References
- [sourceware.org/gdb/current/onlinedocs/gdb.html/Server.html](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Server.html)
- [sourceware.org/gdb/current/onlinedocs/gdb.html/Connecting.html](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Connecting.html)
