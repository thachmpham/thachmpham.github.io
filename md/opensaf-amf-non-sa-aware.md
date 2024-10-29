<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    mermaid.initialize({
        look: 'handDrawn',
        theme: 'neutral',
    });
</script>


---
title:  'OpenSAF AMF'
subtitle: '**Convert a Linux Program to High Availability**'
---


# 1. Introduction.
#### Agenda.
- Create a daemon Linux program.
- Convert the program to be high available using AMF 2N redundancy model. Active on SC-1, standby on SC-2, or vise versa.

#### Prerequisite.
- [Setup a cluster with 2 SCs.](./opensaf-2sc.html)


# 2. The Linux Program.
The program is located at /opt/demo, consists of two scripts:

- /opt/demo/loop.sh
    - Continuous print 'hello' to syslog every 5 seconds
- /opt/demo/main.sh
    - Start, stop the loop.sh script.
    - Check status of the loop.

The program is installed to SC-1 and SC-2.

File /opt/demo/loop.sh.
```sh

counter=1

while true; do
    logger "hello $counter"
    ((counter++))
    sleep 5
done
  
```

File /opt/demo/main.sh.
```sh
  
program='/opt/demo/loop.sh'
pidfile='/var/run/demo.pid'

start() {
    echo 'demo starting...'
    logger 'demo starting...'

    start-stop-daemon --start --background \
        --make-pidfile --pidfile $pidfile \
        --exec /bin/bash -- $program

    echo 'demo started'
    logger 'demo started'
}

stop() {
    echo 'demo stopping...'
    logger 'demo stopping...'

    start-stop-daemon --stop --pidfile $pidfile \
        --retry 5

    echo 'stopped'
    logger 'demo stopped'
}

status() {
    if [ -f $pidfile ] \
        && start-stop-daemon --status --pidfile $pidfile;\
    then
        echo "running, pid: $(cat $pidfile)"
    else
        echo 'not running'
    fi
}

case $1 in
    start)
        start;;
    stop)
        stop;;
    status)
        status;;
    *)
        echo 'Usage: main.sh {start|stop|status}'
        exit 1;;
esac

exit 0
  
```

Permissions.
```sh
  
chmod +x /opt/demo/loop.sh
chmod +x /opt/demo/main.sh
  
```




# 3. Wrap Program by AMF.
#### Agenda.
- Create basetypes: APP, SG, SU, SI, COMP, CSI.
- Create types: APP, SG, SU, SI, COMP, CSI, SwBundle.
- Create associated types: SU-COMP, COMP-CSI, CSI-SI.
- Create common entities: APP, SG, SI, CSI.
- Create entities for SC-1: SU, COMP, NodeSwBundle, COMP-CSI.
- Create entities for SC-2: SU, COMP, NodeSwBundle, COMP-CSI.


## 3.1. AMF BaseTypes.
<pre class="mermaid">
classDiagram
    class SafAmfAppBaseType {
        dn: "safAppType=..."
    }

    class SafAmfSGBaseType {
        dn: "safSgType=..."
    }

    class SafAmfSUBaseType {
        dn: "safSuType=..."
    }
</pre>

<pre class="mermaid">
classDiagram
    class SaAmfSvcBaseType {
        dn: "safSvcType=..."
    }

    class SaAmfCompBaseType {
        dn: "safCompType=..."
    }

    class SaAmfCSBaseType {
        dn: "safCSType=..."
    }
</pre>


```sh
  
# app
$ immcfg -c SaAmfAppBaseType safAppType=demo

# sg
$ immcfg -c SaAmfSGBaseType safSgType=demo

# su
$ immcfg -c SaAmfSUBaseType safSuType=demo

# si
$ immcfg -c SaAmfSvcBaseType safSvcType=demo

# comp
$ immcfg -c SaAmfCompBaseType safCompType=demo

# csi
$ immcfg -c SaAmfCSBaseType safCSType=demo
  
```

## 3.2. AMF Types.
<pre class="mermaid">
classDiagram
    class SaAmfCompType {
        dn: "safSmfBundle=..."
        saAmfCtSwBundle
    }
    SaAmfCompType --> SaSmfSwBundle: saAmfCtSwBundle

    class SaAmfAppType {
        dn: "safVersion=...,safAppType=..."
        saAmfApptSGTypes
    }
    SaAmfAppType --> SaAmfSGType: saAmfApptSGTypes

    class SaAmfSGType {
        dn: "safVersion=...,safSgType=..."
        saAmfSgtValidSuTypes
    }
    SaAmfSGType --> SaAmfSUType: saAmfSgtValidSuTypes

    class SaAmfSUType {
        dn: "safVersion=...,safSuType=..."
        saAmfSutProvidesSvcTypes
    }
    SaAmfSUType --> SaAmfSvcType: saAmfSutProvidesSvcTypes

    class SaSmfSwBundle {
        dn: "safSmfBundle=..."
    }

    class SaAmfSvcType {
        dn: "safVersion=...,safSvcType=..."
    }
</pre>

<pre class="mermaid">
classDiagram
    class SaAmfCSType {
        dn: "safVersion=...,safCSType=..."
    }
</pre>


```sh
  
# swbundle
$ immcfg -c SaSmfSwBundle safSmfBundle=demo

# csi
$ immcfg -c SaAmfCSType safVersion=1,safCSType=demo

# comp
$ immcfg -c SaAmfCompType \
    -a saAmfCtCompCategory=8 \
    -a saAmfCtSwBundle=safSmfBundle=demo \
    -a saAmfCtDefClcCliTimeout=10000000000 \
    -a saAmfCtRelPathInstantiateCmd='main.sh start' \
    -a saAmfCtRelPathCleanupCmd='main.sh stop' \
    -a saAmfCtRelPathTerminateCmd='main.sh stop' \
    -a saAmfCtRelPathAmStartCmd='../../usr/local/sbin/amfpm --start' \
    -a saAmfCtRelPathAmStopCmd='../../usr/local/sbin/amfpm --stop' \
    -a saAmfCtDefRecoveryOnError=3 \
    -a saAmfCtDefDisableRestart=0 \
    safVersion=1,safCompType=demo

# si
$ immcfg -c SaAmfSvcType safVersion=1,safSvcType=demo

# su
$ immcfg -c SaAmfSUType \
    -a saAmfSutIsExternal=0 \
    -a saAmfSutDefSUFailover=1 \
    -a saAmfSutProvidesSvcTypes=safVersion=1,safSvcType=demo \
    safVersion=1,safSuType=demo

# sg
$ immcfg -c SaAmfSGType \
    -a saAmfSgtRedundancyModel=1 \
    -a saAmfSgtValidSuTypes=safVersion=1,safSuType=demo \
    -a saAmfSgtDefAutoAdjustProb=10000000000 \
    -a saAmfSgtDefCompRestartProb=4000000000 \
    -a saAmfSgtDefCompRestartMax=10 \
    -a saAmfSgtDefSuRestartProb=4000000000 \
    -a saAmfSgtDefSuRestartMax=10 \
    safVersion=1,safSgType=demo

# app
$ immcfg -c SaAmfAppType \
    -a saAmfApptSGTypes=safVersion=1,safSgType=demo \
    safVersion=1,safAppType=demo
  
```


## 3.3. AMF Associated Types.
The associated type establishes a connection between two different type using distinguished name (DN). Its DN includes the DNs of the two types it links.
<pre class="mermaid">
classDiagram    
    class SaAmfSutCompType {
        dn: "safMemberCompType=..."
    }
    SaAmfSutCompType --> SaAmfSUType: dn
    SaAmfSutCompType --> SaAmfCompType: dn  

    class SaAmfCtCsType {
        dn: "safSupportedCsType=..."
    }
    SaAmfCtCsType --> SaAmfCompType: dn
    SaAmfCtCsType --> SaAmfCSType: dn

    class SaAmfSvcTypeCSTypes {
        dn: "safMemberCSType=..."
    }
    SaAmfSvcTypeCSTypes --> SaAmfSvcType: dn
    SaAmfSvcTypeCSTypes --> SaAmfCSType: dn
</pre>


```sh
  
# connect su - comp
$ immcfg -c SaAmfSutCompType \
    'safMemberCompType=safVersion=1\,safCompType=demo,safVersion=1,safSuType=demo'

# connect comp - csi
$ immcfg -c SaAmfCtCsType \
    -a saAmfCtCompCapability=1 \
    'safSupportedCsType=safVersion=1\,safCSType=demo,safVersion=1,safCompType=demo'

# connect csi - si
$ immcfg -c SaAmfSvcTypeCSTypes \
    'safMemberCSType=safVersion=1\,safCSType=demo,safVersion=1,safSvcType=demo'
  
```

## 3.4. AMF Common Entities.
Create common AMF entities for both SC-1, SC-2.
<pre class="mermaid">
classDiagram
    class SaAmfApplication {
        dn: "safApp=..."
    }

    class SaAmfSG {
        dn: "safSg=...,safApp=..."
    }
    SaAmfSG --> SaAmfApplication: dn

    class SaAmfSI {
        dn: "safSi=...,safApp=..."
        saAmfSIProtectedbySG
    }
    SaAmfSI --> SaAmfApplication: dn
    SaAmfSI --> SaAmfSG: saAmfSIProtectedbySG

    class SaAmfCSI {
        dn: "safCsi=...,safSi=..."
    }
    SaAmfCSI --> SaAmfSI: dn
</pre>

```sh
  
$ immcfg -c SaAmfApplication \
    -a saAmfAppType=safVersion=1,safAppType=demo \
    safApp=demo

$ immcfg -c SaAmfSG \
    -a saAmfSGType=safVersion=1,safSgType=demo \
    -a saAmfSGAutoRepair=0 \
    -a saAmfSGAutoAdjust=0 \
    -a saAmfSGNumPrefInserviceSUs=10 \
    -a saAmfSGNumPrefAssignedSUs=10 \
    safSg=demo,safApp=demo

$ immcfg -c SaAmfSI \
    -a saAmfSvcType=safVersion=1,safSvcType=demo \
    -a saAmfSIProtectedbySG=safSg=demo,safApp=demo \
    -a saAmfSIRank=1 \
    safSi=demo,safApp=demo

$ immcfg -c SaAmfCSI \
    -a saAmfCSType=safVersion=1,safCSType=demo \
    safCsi=demo,safSi=demo,safApp=demo
  
```

## 3.5. AMF Entities for SC-1.
<pre class="mermaid">
classDiagram
    class SaAmfSU {
        dn: "safSu=...,safSg=..."
        saAmfSUHostNodeOrNodeGroup
    }
    SaAmfSU --> SaAmfNodeGroup: saAmfSUHostNodeOrNodeGroup
    SaAmfSU --> SaAmfNode: saAmfSUHostNodeOrNodeGroup

    class SaAmfNodeGroup {
        dn:"safAmfNode=..."
        safAmfNGNodeList
    }
    SaAmfNodeGroup o-- SaAmfNode: safAmfNGNodeList

    class SaAmfNode {
        dn: "safAmfNode=..."
    }

    class SaAmfComp {
        dn: "safComp=...,safSu=..."
    }
    SaAmfComp --> SaAmfSU: dn
    
    class SaAmfCompCsType {
        dn: "safSupportedCsType=..."
    }
    SaAmfCompCsType --> SaAmfComp: dn
    SaAmfCompCsType --> SaAmfCSType: dn

    class SaAmfCSType {
        dn: "safVersion=...,safCSType=..."
    }

        class SaAmfNodeSwBundle {
        dn: "safInstalledSwBundle=..."
        saAmfNodeSwBundlePathPrefix
    }
    SaAmfNodeSwBundle --> SaAmfNode: dn
</pre>


```sh
    
$ immcfg -c SaAmfNodeSwBundle \
    -a saAmfNodeSwBundlePathPrefix=/opt/demo \
    safInstalledSwBundle=safSmfBundle=demo,safAmfNode=SC-1,safAmfCluster=myAmfCluster

$ immcfg -c SaAmfSU \
    -a saAmfSUType=safVersion=1,safSuType=demo \
    -a saAmfSUHostNodeOrNodeGroup=safAmfNode=SC-1,safAmfCluster=myAmfCluster \
    -a saAmfSURank=1 \
    -a saAmfSUAdminState=3 \
    safSu=1,safSg=demo,safApp=demo

$ immcfg -c SaAmfComp \
    -a saAmfCompType=safVersion=1,safCompType=demo \
    safComp=demo,safSu=1,safSg=demo,safApp=demo

$ immcfg -c SaAmfCompCsType \
    'safSupportedCsType=safVersion=1\,safCSType=demo,safComp=demo,safSu=1,safSg=demo,safApp=demo'
  
```


## 3.6. AMF Entities for SC-2.
```sh
  
$ immcfg -c SaAmfNodeSwBundle \
    -a saAmfNodeSwBundlePathPrefix=/opt/demo \
    safInstalledSwBundle=safSmfBundle=demo,safAmfNode=SC-2,safAmfCluster=myAmfCluster

$ immcfg -c SaAmfSU \
    -a saAmfSUType=safVersion=1,safSuType=demo \
    -a saAmfSUHostNodeOrNodeGroup=safAmfNode=SC-2,safAmfCluster=myAmfCluster \
    -a saAmfSURank=1 \
    -a saAmfSUAdminState=3 \
    safSu=2,safSg=demo,safApp=demo

$ immcfg -c SaAmfComp \
    -a saAmfCompType=safVersion=1,safCompType=demo \
    safComp=demo,safSu=2,safSg=demo,safApp=demo

$ immcfg -c SaAmfCompCsType \
    'safSupportedCsType=safVersion=1\,safCSType=demo,safComp=demo,safSu=2,safSg=demo,safApp=demo'
  
```


# 4. Start Program by AMF.
## 4.1. Unlock SU for SC-1.
Unlock SU for SC-1.
```sh
  
$ amf-adm unlock-in safSu=1,safSg=demo,safApp=demo

$ amf-adm unlock safSu=1,safSg=demo,safApp=demo
  
```

Check `/var/log/syslog`.
```sh
  
SC-1 osafamfnd[12182]: NO Assigning 'safSi=demo,safApp=demo' ACTIVE to 'safSu=1,safSg=demo,safApp=demo'
SC-1 osafamfnd[12182]: NO 'safSu=1,safSg=demo,safApp=demo' Presence State UNINSTANTIATED => INSTANTIATING
SC-1 root: demo starting...
SC-1 root: demo started
SC-1 osafamfnd[12182]: NO 'safSu=1,safSg=demo,safApp=demo' Presence State INSTANTIATING => INSTANTIATED
SC-1 osafamfnd[12182]: NO Assigned 'safSi=demo,safApp=demo' ACTIVE to 'safSu=1,safSg=demo,safApp=demo'
SC-1 root: hello 1
SC-1 root: hello 2
SC-1 root: hello 3
  
```
- As the syslog, we can see that the **demo** program **started successfully**.

Check AMF state.
```sh
  
$ amf-state su
safSu=1,safSg=demo,safApp=demo
        saAmfSUAdminState=UNLOCKED(1)
        saAmfSUOperState=ENABLED(1)
        saAmfSUPresenceState=INSTANTIATED(3)
        saAmfSUReadinessState=IN-SERVICE(2)
safSu=2,safSg=demo,safApp=demo
        saAmfSUAdminState=LOCKED-INSTANTIATION(3)
        saAmfSUOperState=ENABLED(1)
        saAmfSUPresenceState=UNINSTANTIATED(1)
        saAmfSUReadinessState=OUT-OF-SERVICE(1)

$ amf-state si
safSi=demo,safApp=demo
        saAmfSIAdminState=UNLOCKED(1)
        saAmfSIAssignmentState=PARTIALLY_ASSIGNED(3)

$ amf-state comp
safComp=demo,safSu=1,safSg=demo,safApp=demo
        saAmfCompOperState=ENABLED(1)
        saAmfCompPresenceState=INSTANTIATED(3)
        saAmfCompReadinessState=IN-SERVICE(2)
safComp=demo,safSu=2,safSg=demo,safApp=demo
        saAmfCompOperState=ENABLED(1)
        saAmfCompPresenceState=UNINSTANTIATED(1)
        saAmfCompReadinessState=OUT-OF-SERVICE(1)

$ amf-state csiass
safCSIComp=safComp=demo\,safSu=1\,safSg=demo\,safApp=demo,safCsi=demo,safSi=demo,safApp=demo
        saAmfCSICompHAState=ACTIVE(1)
        saAmfCSICompHAReadinessState=READY_FOR_ASSIGNMENT(1)
  
```


## 4.2. Unlock SU for SC-2.
Unlock SU for SC-2.
```sh
  
$ amf-adm unlock-in safSu=2,safSg=demo,safApp=demo

$ amf-adm unlock safSu=2,safSg=demo,safApp=demo
  
```


Check `/var/log/syslog`.
```sh
  
Oct 29 05:25:29 SC-2 osafamfnd[185]: NO Assigning 'safSi=demo,safApp=demo' STANDBY to 'safSu=2,safSg=demo,safApp=demo'
Oct 29 05:25:29 SC-2 osafamfnd[185]: NO Assigned 'safSi=demo,safApp=demo' STANDBY to 'safSu=2,safSg=demo,safApp=demo'
  
```

Check AMF state.
```sh
  
$ amf-state su
safSu=1,safSg=demo,safApp=demo
        saAmfSUAdminState=UNLOCKED(1)
        saAmfSUOperState=ENABLED(1)
        saAmfSUPresenceState=INSTANTIATED(3)
        saAmfSUReadinessState=IN-SERVICE(2)
safSu=2,safSg=demo,safApp=demo
        saAmfSUAdminState=UNLOCKED(1)
        saAmfSUOperState=ENABLED(1)
        saAmfSUPresenceState=UNINSTANTIATED(1)
        saAmfSUReadinessState=IN-SERVICE(2)

$ amf-state si
safSi=demo,safApp=demo
        saAmfSIAdminState=UNLOCKED(1)
        saAmfSIAssignmentState=FULLY_ASSIGNED(2)

$ amf-state comp
safComp=demo,safSu=1,safSg=demo,safApp=demo
        saAmfCompOperState=ENABLED(1)
        saAmfCompPresenceState=INSTANTIATED(3)
        saAmfCompReadinessState=IN-SERVICE(2)
safComp=demo,safSu=2,safSg=demo,safApp=demo
        saAmfCompOperState=ENABLED(1)
        saAmfCompPresenceState=UNINSTANTIATED(1)
        saAmfCompReadinessState=IN-SERVICE(2)

$ amf-state csiass
safCSIComp=safComp=demo\,safSu=2\,safSg=demo\,safApp=demo,safCsi=demo,safSi=demo,safApp=demo
        saAmfCSICompHAState=STANDBY(2)
        saAmfCSICompHAReadinessState=READY_FOR_ASSIGNMENT(1)
safCSIComp=safComp=demo\,safSu=1\,safSg=demo\,safApp=demo,safCsi=demo,safSi=demo,safApp=demo
        saAmfCSICompHAState=ACTIVE(1)
        saAmfCSICompHAReadinessState=READY_FOR_ASSIGNMENT(1)
  
```

# 5. Fault Tolerance.
If stop the program on SC-1, AMF will restart it on SC-2.

On SC-1, stop the program.
```sh
  
$ /opt/demo/main.sh stop
  
```

Check syslog of SC-1.
```sh
  
SC-1 root: demo stopping...
SC-1 osafamfnd[16640]: NO saAmfSUFailover is true for 'safSu=1,safSg=demo,safApp=demo'
SC-1 osafamfnd[16640]: NO Performing failover of 'safSu=1,safSg=demo,safApp=demo' (SU failover count: 2)
SC-1 osafamfnd[16640]: NO 'safComp=demo,safSu=1,safSg=demo,safApp=demo' recovery action escalated from 'noRecommendation' to 'suFailover'
SC-1 osafamfnd[16640]: NO 'safComp=demo,safSu=1,safSg=demo,safApp=demo' faulted due to 'passiveMonitorFailed' : Recovery is 'suFailover'
SC-1 osafamfnd[16640]: NO Terminating components of 'safSu=1,safSg=demo,safApp=demo'(abruptly & unordered)
SC-1 root: demo stopped
SC-1 osafamfnd[16640]: NO 'safSu=1,safSg=demo,safApp=demo' Presence State INSTANTIATED => TERMINATING
SC-1 osafamfnd[16640]: NO 'safSu=1,safSg=demo,safApp=demo' Presence State TERMINATING => TERMINATING
SC-1 root: demo stopping...
SC-1 root: demo stopped
SC-1 osafamfnd[16640]: NO Terminated all components in 'safSu=1,safSg=demo,safApp=demo'
SC-1 osafamfnd[16640]: NO Informing director of sufailover
SC-1 osafamfnd[16640]: NO 'safSu=1,safSg=demo,safApp=demo' Presence State TERMINATING => UNINSTANTIATED
  
```
- The **demo** program is **stopped on SC-1**.


Check syslog of SC-2.
```sh
  
C-2 osafamfnd[1028]: NO Assigning 'safSi=demo,safApp=demo' ACTIVE to 'safSu=2,safSg=demo,safApp=demo'
C-2 rsyslogd-2007: action 'action 9' suspended, next retry is Tue Oct 29 06:11:50 2024 [v8.16.0 try http://www.rsyslog.com/e/2007 ]
C-2 osafamfnd[1028]: NO 'safSu=2,safSg=demo,safApp=demo' Presence State UNINSTANTIATED => INSTANTIATING
C-2 root: demo starting...
C-2 root: demo started
C-2 osafamfnd[1028]: NO 'safSu=2,safSg=demo,safApp=demo' Presence State INSTANTIATING => INSTANTIATED
C-2 osafamfnd[1028]: NO Assigned 'safSi=demo,safApp=demo' ACTIVE to 'safSu=2,safSg=demo,safApp=demo'
C-2 root: hello 1
SC-2 root: hello 2
SC-2 root: hello 3
  
```
- The **demo** program is **restarted successfully on SC-2**.


Check AMF state.
```sh
  
$ amf-state su
safSu=1,safSg=demo,safApp=demo
        saAmfSUAdminState=UNLOCKED(1)
        saAmfSUOperState=DISABLED(2)
        saAmfSUPresenceState=UNINSTANTIATED(1)
        saAmfSUReadinessState=OUT-OF-SERVICE(1)
safSu=2,safSg=demo,safApp=demo
        saAmfSUAdminState=UNLOCKED(1)
        saAmfSUOperState=ENABLED(1)
        saAmfSUPresenceState=INSTANTIATED(3)
        saAmfSUReadinessState=IN-SERVICE(2)
  
```

# References
- https://opensaf.sourceforge.io/documentation.html
- SAI-AIS-AMF-B.04.01.AL, 8.3 DN Formats for AMF UML Classes