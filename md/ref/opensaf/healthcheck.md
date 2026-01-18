---
title:  'OpenSAF AMF Healthcheck'
---


# 1. Introduction
OpenSAF includes a healthcheck mechanism to monitor the health of processes, detect failures, and initiate recovery actions such as restarting processes or failing over to another node. This ensures high availability and enhances fault tolerance within the system.

Key classes:

- `SaAmfHealthcheckType`
- `SaAmfHealthcheck`

Key Attributes:

- `safHealthcheckKey`
- `saAmfHctDefPeriod`, `saAmfHealthcheckPeriod`: The time interval at which the AMF sends a health check ping to the program. If there is no response within this period, AMF considers the program unresponsive and triggers a restart.
- `saAmfHctDefMaxDuration`, `saAmfHealthcheckMaxDuration`


# 2. Lab
In the [previous post](../html/opensaf-amf-sa-aware.html), we implemented an AMF SA-aware program. Today, we will integrate a healthcheck into the program.

## 2.1. Healthcheck IMM Object
- Create healthcheck imm object.
```sh
  
$ immcfg -c SaAmfHealthcheck \
    -a saAmfHealthcheckPeriod=5000000000 \
    -a saAmfHealthcheckMaxDuration=3000000000 \
    safHealthcheckKey=demo,safComp=demo,safSu=SC-1,safSg=demo,safApp=demo


# immcfg -c SaAmfHealthcheckType \
#     -a saAmfHctDefPeriod=10000000000 \
#     -a saAmfHctDefMaxDuration=5000000000 \
#     safHealthcheckKey=demo,safVersion=1,safCompType=demo

# the time unit is nanosecond
  
```
  
## 2.2. Start Healthcheck
- Edit file main.c.
```c++
  
void healthcheckCallback(SaInvocationT invocation,
    const SaNameT *compName, SaAmfHealthcheckKeyT *key);

int healthcheckCount = 0;

int main(int argc, char ** argv)
{
    // ...
    callbacks.saAmfHealthcheckCallback = healthcheckCallback;
    
    saAmfInitialize_o4(&amfHandler, &callbacks, &apiVersion);

    // start healthcheck
    saAmfHealthcheckStart(
	    amfHandler, &name, &healthcheckKey,
	    SA_AMF_HEALTHCHECK_AMF_INVOKED, SA_AMF_COMPONENT_RESTART);
    // ...
}

void healthcheckCallback(SaInvocationT invocation,
    const SaNameT *compName, SaAmfHealthcheckKeyT *key)
{
    trace("count = %d", healthcheckCount);
    healthcheckCount += 1;

    saAmfResponse_4(amfHandler, invocation, 0, SA_AIS_OK);
}
  
```


## 2.3. Start Service Unit
- Instantiate service unit.
```sh
  
$ amf-adm unlock-in safSu=SC-1,safSg=demo,safApp=demo
  
```

```sh
  
# syslog
2024-12-26T10:50:00.608935-05:00 SC-1 osafamfnd[15994]: NO 'safSu=SC-1,safSg=demo,safApp=demo' Presence State UNINSTANTIATED => INSTANTIATING
2024-12-26T10:50:00.613154-05:00 SC-1 demo[17859]: main: start
2024-12-26T10:50:00.614284-05:00 SC-1 osafamfnd[15994]: NO 'safSu=SC-1,safSg=demo,safApp=demo' Presence State INSTANTIATING => INSTANTIATED
2024-12-26T10:50:00.614606-05:00 SC-1 demo[17859]: main: receive poll event
2024-12-26T10:50:00.614704-05:00 SC-1 demo[17859]: healthcheckCallback: count = 0
2024-12-26T10:50:10.663296-05:00 SC-1 demo[17859]: main: receive poll event
2024-12-26T10:50:10.663724-05:00 SC-1 demo[17859]: healthcheckCallback: count = 1
2024-12-26T10:50:20.761590-05:00 SC-1 demo[17859]: main: receive poll event
2024-12-26T10:50:20.761904-05:00 SC-1 demo[17859]: healthcheckCallback: count = 2
  
```

- Unlock service unit.
```sh
  
$ amf-adm unlock safSu=SC-1,safSg=demo,safApp=demo
  
```

```sh
  
# syslog
2024-12-26T10:51:55.920498-05:00 SC-1 osafamfnd[15994]: NO Assigning 'safSi=demo,safApp=demo' ACTIVE to 'safSu=SC-1,safSg=demo,safApp=demo'
2024-12-26T10:51:55.920990-05:00 SC-1 demo[17859]: main: receive poll event
2024-12-26T10:51:55.921097-05:00 SC-1 demo[17859]: csiSetCallback: 
2024-12-26T10:51:55.921154-05:00 SC-1 osafamfnd[15994]: NO Assigned 'safSi=demo,safApp=demo' ACTIVE to 'safSu=SC-1,safSg=demo,safApp=demo'
  
```


# References
- https://opensaf.sourceforge.io/documentation.html
