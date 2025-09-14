---
title:  Error Detection & Recovery in AMF
---

# 1. Error Detection
In OpenSAF AMF, component is the smallest logical entity performs error detection and recovery.

Three types of component monitoring:

- Passive Monitoring
- External Active Monitoring
- Internal Active Monitoring.


## 1.1. Passive Monitoring
AMF monitors the death of processes that are part of the component. If processes terminated, AMF will automatically report an error on the related component.

APIs:

- `saAmfPmStart_3()`: Start passive monitoring on a process and to its children.
- `saAmfPmStop()`: Stop passive monitoring.


## 1.2. External Active Monitoring
AMF monitors the health of the component by submitting some service requests to the component and checking that the service is provide in a timely fashion.

IMM:

- `AM_START`: start a monitoring process for a component.
- `AM_STOP`: stop a monitoring process for a component.

## 1.3. Internal Active Monitoring
The component includes code to monitor its own health and to discover latent faults. Each of these health checks is triggered either by the component itself or by AMF.

IMM:

- `safHealthcheckKey`: Healthcheck key.
- `saAmfHealthcheckPeriod`, `saAmfHctDefPeriod`: The period healthcheck should be initiated.
- `saAmfHealthcheckMaxDuration`, `saAmfHtcDefMaxDuration`: Time-limit after which AMF will report an error on the component if no response for the healthcheck.

APIs:

- `saAmfHealthcheckStart()`: Start healthcheck on a process.
- `saAmfHealthcheckStop()`: Stop healthcheck on a process.
- `saAmfHealthcheckCallback()`: This callback will be periodically called by AMF to check if process is healthy.
- `saAmfResponse_4()`: Process replies its status to AMF.


# 2. Recovery
Recovery is an automatic action taken by AMF after an error occurred.

IMM:

- `saAmfCompRecoveryOnError`: Recovery action.

API:
```c
  
// File: opensaf/src/ais/include/saAmf.h

typedef enum {
    SA_AMF_NO_RECOMMENDATION = 1,
    SA_AMF_COMPONENT_RESTART = 2,
    SA_AMF_COMPONENT_FAILOVER = 3,
    SA_AMF_NODE_SWITCHOVER = 4,
    SA_AMF_NODE_FAILOVER = 5,
    SA_AMF_NODE_FAILFAST = 6,
    SA_AMF_CLUSTER_RESET = 7,
    SA_AMF_APPLICATION_RESTART = 8,
    SA_AMF_CONTAINER_RESTART = 9
} SaAmfRecommendedRecoveryT;
  
```

- `SA_AMF_NO_RECOMMENDATION`:     Use the default recovery action of AMF.
- `SA_AMF_COMPONENT_RESTART`:     Restart the erroreous component. Objective is to avoid reassigning service instances to different service units. Fix the problem by restart the component and reassign to the same service unit.
- `SA_AMF_COMPONENT_FAILOVER`:    Reassign service instances to other service units.
- `SA_AMF_NODE_SWITCHOVER`:       
- `SA_AMF_NODE_FAILOVER`:         Reassign all service instances of service units on the node to other nodes.
- `SA_AMF_NODE_FAILFAST`:         Reboot the node.
- `SA_AMF_CLUSTER_RESET`:         Reboot all nodes in the cluster.
- `SA_AMF_APPLICATION_RESTART`:   Teminate and restart all service units of the application.
- `SA_AMF_CONTAINER_RESTART`:     


# References
- [SAI AIS AMF, 3.10, Component Monitoring](https://opensaf.sourceforge.io/SAI-AIS-AMF-B.04.01.AL.pdf)
- [SAI AIS AMF, 3.11, Error Detection, Recovery, Repair and Escalation Policy](https://opensaf.sourceforge.io/SAI-AIS-AMF-B.04.01.AL.pdf)
- [SAI AIS AMF, 4.9, AM_START](https://opensaf.sourceforge.io/SAI-AIS-AMF-B.04.01.AL.pdf)
- [SAI AIS AMF, 7.1.2, Component Healthcheck Monitoring](https://opensaf.sourceforge.io/SAI-AIS-AMF-B.04.01.AL.pdf)
- [SAI AIS AMF, 7.7.1, saAmfPmStart_3](https://opensaf.sourceforge.io/SAI-AIS-AMF-B.04.01.AL.pdf)