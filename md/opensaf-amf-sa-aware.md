---
title:  'OpenSAF AMF'
subtitle: 'Build High Availability App with AMF API'
---


# 1. Introduction
#### Agenda
- Create an AMF program using OpenSAF API.
- Integrate the program to IMM.
- Start, stop program and failure tolerance.

# 2. The AMF Program
The program is located at /opt/demo, consists of two files:

- /opt/demo/main.c
    - Register and listen AMF callbacks.
- /opt/demo/control.sh
    - Start, stop the program.

The program will be installed to SC-1.

File /opt/demo/main.c
```C
  
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <stdlib.h>
#include <unistd.h>
#include <poll.h>
#include <syslog.h>
#include <signal.h>
#include <saAmf.h>
#include <saAis.h>


#define trace(fmt, ...) \
    syslog(LOG_INFO, "%s: " fmt, __FUNCTION__, ##__VA_ARGS__)


void csiSetCallback(SaInvocationT invocation, const SaNameT *name,
    SaAmfHAStateT state, SaAmfCSIDescriptorT descriptor);

void csiRemoveCallback(SaInvocationT invocation, const SaNameT *compName,
	const SaNameT *csiName, SaAmfCSIFlagsT csiFlags);

void componentTerminateCallback(SaInvocationT invocation,
	const SaNameT *compName);

void csiAttributeChangeCallback(SaInvocationT invocation,
    const SaNameT *csiName, SaAmfCSIAttributeListT csiAttr);


SaAmfHandleT amfHandler;


int main(int argc, char ** argv)
{
    // setup syslog
    openlog("demo", LOG_PID, LOG_USER);
    trace("start");

    // check if program start by AMF
    // SA_AMF_COMPONENT_NAME exists when started by amf
    if (getenv("SA_AMF_COMPONENT_NAME") == NULL)
    {
        trace("not started by AMF, exiting...\n");
        return 1;
    }

    // setup amf callbacks
    SaAmfCallbacksT_o4 callbacks = {0};
    callbacks.saAmfCSISetCallback = csiSetCallback;
    callbacks.saAmfCSIRemoveCallback = csiRemoveCallback;
    callbacks.saAmfComponentTerminateCallback= componentTerminateCallback;
    callbacks.osafCsiAttributeChangeCallback = csiAttributeChangeCallback;
    SaVersionT apiVersion = {
		.releaseCode = 'B',
        .majorVersion = 0x04,
		.minorVersion = 0x02
    };
    saAmfInitialize_o4(&amfHandler, &callbacks, &apiVersion);

    // register component
	SaNameT name;
	saAmfComponentNameGet(amfHandler, &name);
    saAmfComponentRegister(amfHandler, &name, 0);

    // get file descriptor that listens incoming amf callbacks
    SaSelectionObjectT monitorFd;
    saAmfSelectionObjectGet(amfHandler, &monitorFd);

    // setup pollfd to monitor amf callback events
    struct pollfd fds[1];
    fds[0].fd = monitorFd;
    fds[0].events = POLLIN;

    // wait and handle amf callback events
    while (poll(fds, 1, -1) != -1)
    {
        if (fds[0].revents & POLLIN)
        {
            trace("receive poll event");
            saAmfDispatch(amfHandler, SA_DISPATCH_ONE);
        }
    }

    return 1;
}


void csiSetCallback(SaInvocationT invocation, const SaNameT *name,
    SaAmfHAStateT state, SaAmfCSIDescriptorT descriptor)
{
    trace();
    saAmfResponse_4(amfHandler, invocation, 0, SA_AIS_OK);
}


void csiRemoveCallback(SaInvocationT invocation, const SaNameT *compName,
    const SaNameT *csiName, SaAmfCSIFlagsT csi_flags)
{
    trace();
    saAmfResponse_4(amfHandler, invocation, 0, SA_AIS_OK);
}


void componentTerminateCallback(SaInvocationT invocation,
	const SaNameT *compName)
{
    trace();
    saAmfResponse_4(amfHandler, invocation, 0, SA_AIS_OK);
    exit(0);
}


void csiAttributeChangeCallback(SaInvocationT invocation,
    const SaNameT *csiName, SaAmfCSIAttributeListT csiAttr)
{
    trace();
    saAmfResponse_4(amfHandler, invocation, 0, SA_AIS_OK);
}
  
```

Compile.
```sh
  
$ gcc -g -o main main.c -lSaAmf
  
```

File /opt/demo/control.h
```sh
  
start() {
    logger -st demo 'start'
    start-stop-daemon --start --background \
        --make-pidfile --pidfile /var/run/demo.pid \
        --exec /opt/demo/main
}

stop() {
    logger -st demo 'stop'
    start-stop-daemon --stop --pidfile /var/run/demo.pid
}

case $1 in
    start)
        start;;
    stop)
        stop;;
    *)
        echo 'Usage: control.sh {start|stop}'
        exit 1;;
esac

exit 0
  
```


# 3. Software Bundle
```sh
  
immcfg -c SaSmfSwBundle safSmfBundle=demo

immcfg -c SaAmfNodeSwBundle \
    -a saAmfNodeSwBundlePathPrefix=/opt/demo \
    safInstalledSwBundle=saSmfBundle=demo,safAmfNode=SC-1,safAmfCluster=myAmfCluster
  
```


# 4. Base Types
```sh
  
immcfg -c SaAmfAppBaseType safAppType=demo

immcfg -c SaAmfSGBaseType safSgType=demo

immcfg -c SaAmfSUBaseType safSuType=demo

immcfg -c SaAmfSvcBaseType safSvcType=demo

immcfg -c SaAmfCompBaseType safCompType=demo

immcfg -c SaAmfCSBaseType safCSType=demo
  
```


# 5. Types
```sh
  
immcfg -c SaAmfCSType safVersion=1,safCSType=demo

immcfg -c SaAmfSvcType safVersion=1,safSvcType=demo

immcfg -c SaAmfCompType \
    -a saAmfCtCompCategory=1 \
    -a saAmfCtSwBundle=saSmfBundle=demo \
    -a saAmfCtDefClcCliTimeout=10000000000 \
    -a saAmfCtDefCallbackTimeout=10000000000 \
    -a saAmfCtRelPathInstantiateCmd=control.sh \
    -a saAmfCtDefInstantiateCmdArgv=start \
    -a saAmfCtRelPathCleanupCmd=control.sh \
    -a saAmfCtDefCleanupCmdArgv=stop \
    -a saAmfCtDefQuiescingCompleteTimeout=10000000000 \
    -a saAmfCtDefRecoveryOnError=2 \
    -a saAmfCtDefDisableRestart=0 \
    safVersion=1,safCompType=demo

immcfg -c SaAmfSUType \
    -a saAmfSutProvidesSvcTypes=safVersion=1,safSvcType=demo \
    -a saAmfSutIsExternal=0 \
    -a saAmfSutDefSUFailover=1 \
    safVersion=1,safSuType=demo

immcfg -c SaAmfSGType \
    -a saAmfSgtValidSuTypes=safVersion=1,safSuType=demo \
    -a saAmfSgtRedundancyModel=1 \
    -a saAmfSgtDefSuRestartProb=4000000000 \
    -a saAmfSgtDefSuRestartMax=3 \
    -a saAmfSgtDefCompRestartProb=4000000000 \
    -a saAmfSgtDefCompRestartMax=3 \
    -a saAmfSgtDefAutoAdjustProb=10000000000 \
    safVersion=1,safSgType=demo

immcfg -c SaAmfAppType \
    -a saAmfApptSGTypes=safVersion=1,safSgType=demo \
    safVersion=1,safAppType=demo
  
```


# 6. Associated Types
```sh
  
immcfg -c SaAmfSutCompType \
    'safMemberCompType=safVersion=1\,safCompType=demo,safVersion=1,safSuType=demo'

immcfg -c SaAmfCtCsType \
    -a saAmfCtCompCapability=1 \
    'safSupportedCsType=safVersion=1\,safCSType=demo,safVersion=1,safCompType=demo'


immcfg -c SaAmfSvcTypeCSTypes \
    'safMemberCSType=safVersion=1\,safCSType=demo,safVersion=1,safSvcType=demo'
  
```


# 7. Common Entities
```sh
  
immcfg -c SaAmfApplication \
    -a saAmfAppType=safVersion=1,safAppType=demo \
    safApp=demo

immcfg -c SaAmfSG \
    -a saAmfSGType=safVersion=1,safSgType=demo \
    -a saAmfSGSuHostNodeGroup=safAmfNodeGroup=SCs,safAmfCluster=myAmfCluster \
    -a saAmfSGAutoRepair=0 \
    -a saAmfSGAutoAdjust=0 \
    -a saAmfSGNumPrefInserviceSUs=10 \
    -a saAmfSGNumPrefAssignedSUs=10 \
    safSg=demo,safApp=demo

immcfg -c SaAmfSI \
    -a saAmfSvcType=safVersion=1,safSvcType=demo \
    -a saAmfSIProtectedbySG=safSg=demo,safApp=demo \
    safSi=demo,safApp=demo

immcfg -c SaAmfCSI \
    -a saAmfCSType=safVersion=1,safCSType=demo \
    safCsi=demo,safSi=demo,safApp=demo
  
```


# 8. Entities for SC-1
```sh
  
immcfg -c SaAmfSU \
    -a saAmfSUType=safVersion=1,safSuType=demo \
    -a saAmfSURank=1 \
    -a saAmfSUAdminState=3 \
    safSu=SC-1,safSg=demo,safApp=demo

immcfg -c SaAmfComp \
    -a saAmfCompType=safVersion=1,safCompType=demo \
    safComp=demo,safSu=SC-1,safSg=demo,safApp=demo

immcfg -c SaAmfCompCsType \
    'safSupportedCsType=safVersion=1\,safCSType=demo,safComp=demo,safSu=SC-1,safSg=demo,safApp=demo'
  
```


# 9. Start Program
Unlock instantiate (start process).
```sh
  
amf-adm unlock-in safSu=SC-1,safSg=demo,safApp=demo
  
```

```sh
  
# syslog
SC-1 osafamfnd[8729]: NO 'safSu=SC-1,safSg=demo,safApp=demo' Presence State UNINSTANTIATED => INSTANTIATING
SC-1 demo: start
SC-1 demo[13044]: main: start
SC-1 osafamfnd[8729]: NO 'safSu=SC-1,safSg=demo,safApp=demo' Presence State INSTANTIATING => INSTANTIATED
  
```


Unlock SU (assign work to process).
```sh
  
amf-adm unlock safSu=SC-1,safSg=demo,safApp=demo
  
```

```sh
  
# syslog
SC-1 osafamfnd[8729]: NO Assigning 'safSi=demo,safApp=demo' ACTIVE to 'safSu=SC-1,safSg=demo,safApp=demo'
SC-1 demo[13044]: main: receive poll event
SC-1 demo[13044]: csiSetCallback: 
SC-1 osafamfnd[8729]: NO Assigned 'safSi=demo,safApp=demo' ACTIVE to 'safSu=SC-1,safSg=demo,safApp=demo'
  
```


# 10. Fault Tolerance
Kill the process.
```sh
  
/opt/demo/control.sh stop
  
```

The program will be restarted automatically by AMF.
```sh
  
# syslog
SC-1 demo: stop
SC-1 osafamfnd[8729]: NO 'safSu=SC-1,safSg=demo,safApp=demo' component restart probation timer started (timeout: 4000000000 ns)
SC-1 osafamfnd[8729]: NO Restarting a component of 'safSu=SC-1,safSg=demo,safApp=demo' (comp restart count: 1)
SC-1 osafamfnd[8729]: NO 'safComp=demo,safSu=SC-1,safSg=demo,safApp=demo' faulted due to 'avaDown' : Recovery is 'componentRestart'
SC-1 osafamfnd[8729]: NO 'safSu=SC-1,safSg=demo,safApp=demo' Presence State INSTANTIATED => RESTARTING
SC-1 demo: stop
SC-1 demo: start
SC-1 demo[13185]: main: start
SC-1 osafamfnd[8729]: NO 'safSu=SC-1,safSg=demo,safApp=demo' Presence State RESTARTING => INSTANTIATED
SC-1 demo[13185]: main: receive poll event
SC-1 demo[13185]: csiSetCallback: 
SC-1 osafamfnd[8729]: NO 'safSu=SC-1,safSg=demo,safApp=demo' Component or SU restart probation timer expired
  
```

# 11. Stop Program
Lock SU (remove work from process).
```sh
  
amf-adm lock safSu=SC-1,safSg=demo,safApp=demo
  
```

```sh
  
# syslog
SC-1 osafamfnd[8729]: NO Assigning 'safSi=demo,safApp=demo' QUIESCED to 'safSu=SC-1,safSg=demo,safApp=demo'
SC-1 demo[13185]: main: receive poll event
SC-1 demo[13185]: csiSetCallback: 
SC-1 osafamfnd[8729]: NO Assigned 'safSi=demo,safApp=demo' QUIESCED to 'safSu=SC-1,safSg=demo,safApp=demo'
SC-1 osafamfnd[8729]: NO Removing 'safSi=demo,safApp=demo' from 'safSu=SC-1,safSg=demo,safApp=demo'
SC-1 demo[13185]: main: receive poll event
SC-1 demo[13185]: csiRemoveCallback: 
SC-1 osafamfnd[8729]: NO Removed 'safSi=demo,safApp=demo' from 'safSu=SC-1,safSg=demo,safApp=demo'

```

Lock instantiate (stop process).
```sh
  
amf-adm lock-in safSu=SC-1,safSg=demo,safApp=demo
  
```

```sh
  
# syslog
SC-1 osafamfnd[8729]: NO 'safSu=SC-1,safSg=demo,safApp=demo' component restart probation timer stopped
SC-1 osafamfnd[8729]: NO 'safSu=SC-1,safSg=demo,safApp=demo' Presence State INSTANTIATED => TERMINATING
SC-1 demo[13185]: main: receive poll event
SC-1 demo[13185]: componentTerminateCallback: 
SC-1 demo: stop
SC-1 osafamfnd[8729]: NO 'safSu=SC-1,safSg=demo,safApp=demo' Presence State TERMINATING => UNINSTANTIATED
  
```

# References
- https://opensaf.sourceforge.io/documentation.html
