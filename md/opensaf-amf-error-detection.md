---
title: Error Detection in OpenSAF AMF
---

# 1. Component Monitoring
- **Passive Monitoring**
    - AMF monitors the death of processes that are part of the component.
    - `saAmfPmStart_3()`: requests AMF to start passive monitoring on a process and to its descendents. If a process termination occurs, AMF will automatically report an error on the related component.
    - `saAmfPmStop()`: requests AMF to stop passive monitoring.

- **External Active Monitoring**
    - AMF monitors the health of the component by submitting some service requests to the component and checking that the service is provide in a timely fashion.
    - `AM_START`: start a monitoring process for a component.
    - `AM_STOP`: stop a monitoring process for a component.

- **Internal Active Monitoring**
    - The component includes code to monitor its own health and to discover latent faults. Each of these health checks is triggered either by the component itself or by AMF.
    - `saAmfHealthcheckStart()`: Request to start healthcheck on a process.
    - `saAmfHealthcheckStop()`: Request to stop healthcheck on a process.
    - `saAmfHealthcheckCallback()`: This callback will be periodically called by AMF.
    - `saAmfResponse_4()`: Process to reply its status to AMF.
    - `safHealthcheckKey`: Healthcheck key.
    - `saAmfHealthcheckPeriod`, `saAmfHctDefPeriod`: the period healthcheck should be initiated.
    - `saAmfHealthcheckMaxDuration`, `saAmfHtcDefMaxDuration`: time-limit after which AMF will report an error on the component if no response for the healthcheck.


# References
- [SAI AIS AMF, 3.10, Component Monitoring](https://opensaf.sourceforge.io/SAI-AIS-AMF-B.04.01.AL.pdf)
- [SAI AIS AMF, 7.7.1, saAmfPmStart_3](https://opensaf.sourceforge.io/SAI-AIS-AMF-B.04.01.AL.pdf)
- [SAI AIS AMF, 4.9, AM_START](https://opensaf.sourceforge.io/SAI-AIS-AMF-B.04.01.AL.pdf)
- [SAI AIS AMF, 7.1.2, Component Healthcheck Monitoring](https://opensaf.sourceforge.io/SAI-AIS-AMF-B.04.01.AL.pdf)