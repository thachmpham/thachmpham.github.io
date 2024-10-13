---
title:  'OpenSAF LOG'
---


# 1. Introduction.
In OpenSAF, the Log Service (LOGSv) provides a standardized means for applications to write log messages.
  
There are two options to configure LOGSv:

- Using IMM object.
- Using environment variables (deprecated).

The `saflogger` program is used to write OpenSAF log. 


# 2. Configure.
## 2.1. Using IMM object.
The configurations can be specified through an object of `OpenSafLogConfig` class. The object name is `logConfig=1,safApp=safLogService`.

```s
   
$ immfind -c OpenSafLogConfig
logConfig=1,safApp=safLogService

$ immlist logConfig=1,safApp=safLogService
Name                                               Type         Value(s)
========================================================================
logStreamSystemLowLimit                            SA_UINT32_T  0 (0x0)
logStreamSystemHighLimit                           SA_UINT32_T  0 (0x0)
logStreamFileFormat                                SA_STRING_T  <Empty>
logStreamAppLowLimit                               SA_UINT32_T  0 (0x0)
logStreamAppHighLimit                              SA_UINT32_T  0 (0x0)
logRootDirectory                                   SA_STRING_T  /var/log/opensaf/saflog 
logRecordDestinationConfiguration                  SA_STRING_T  <Empty>
logMaxLogrecsize                                   SA_UINT32_T  1024 (0x400)
logMaxApplicationStreams                           SA_UINT32_T  64 (0x40)
logFileSysConfig                                   SA_UINT32_T  1 (0x1)
logFileIoTimeout                                   SA_UINT32_T  500 (0x1f4)
logDataGroupname                                   SA_STRING_T  <Empty>
logConfig                                          SA_STRING_T  logConfig=1 
SaImmAttrImplementerName                           SA_STRING_T  safLogService 
SaImmAttrClassName                                 SA_STRING_T  OpenSafLogConfig 
SaImmAttrAdminOwnerName                            SA_STRING_T  IMMLOADER
  
```
- `logRootDirectory` must be an existing directory with read and write access.

## 2.1. Using environment variable.
The configurations can be set in the `/etc/opensaf/logd.conf` file.
```sh
  
export LGSV_ENV_HEALTHCHECK_KEY="Default"
export LOGSV_ROOT_DIRECTORY=$pkglogdir/saflog
export LOGSV_MAX_LOGRECSIZE=1024
args="--loglevel=info"
  
```

# 3. saflogger.
Make an alarm log message.
```sh
  
$ saflogger --alarm "hello alarm"

$ grep -r "hello alarm" /var/log/opensaf/saflog
/var/log/opensaf/saflog/saLogAlarm_20241013_140309.log: ... "hello alarm"
  
```

Make a system log message.
```sh
  
$ saflogger --system "hello system"
  
```

Make an application log message.
```sh
  
$ saflogger --application -f myApp "hello application"
/var/log/opensaf/saflog/saflogger/TestApp_20241013_150159_20241013_150159.log: ... "hello application"
  
```


# References
- https://opensaf.sourceforge.io/documentation.html