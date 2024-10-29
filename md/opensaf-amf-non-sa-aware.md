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


# 2. Create Linux Program.
Create directory `/opt/demo`.
```sh
  
$ mkdir /opt/demo
  
```

Create script `/opt/demo/loop.sh` that continuous print 'hello' to syslog every 5 seconds.
```sh

counter=1

while true; do
    logger "hello $counter"
    ((counter++))
    sleep 5
done
  
```

Create script `/opt/demo/main.sh` to manage the loop, allowing to start, stop and check status of the loop.
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

Grant permissions.
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
immcfg -c SaAmfSutCompType \
    'safMemberCompType=safVersion=1\,safCompType=demo,safVersion=1,safSuType=demo'

# connect comp - csi
immcfg -c SaAmfCtCsType \
    -a saAmfCtCompCapability=1 \
    'safSupportedCsType=safVersion=1\,safCSType=demo,safVersion=1,safCompType=demo'

# connect csi - si
immcfg -c SaAmfSvcTypeCSTypes \
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
  
immcfg -c SaAmfApplication \
    -a saAmfAppType=safVersion=1,safAppType=demo \
    safApp=demo

immcfg -c SaAmfSG \
    -a saAmfSGType=safVersion=1,safSgType=demo \
    -a saAmfSGAutoRepair=0 \
    -a saAmfSGAutoAdjust=0 \
    -a saAmfSGNumPrefInserviceSUs=10 \
    -a saAmfSGNumPrefAssignedSUs=10 \
    safSg=demo,safApp=demo

immcfg -c SaAmfSI \
    -a saAmfSvcType=safVersion=1,safSvcType=demo \
    -a saAmfSIProtectedbySG=safSg=demo,safApp=demo \
    -a saAmfSIRank=1 \
    safSi=demo,safApp=demo

immcfg -c SaAmfCSI \
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

$ immcfg -c SaAmfNodeSwBundle \
    -a saAmfNodeSwBundlePathPrefix=/opt/demo \
    safInstalledSwBundle=safSmfBundle=demo,safAmfNode=SC-1,safAmfCluster=myAmfCluster
  
```


## 3.6. AMF Entities for SC-2.
```sh
  
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

$ immcfg -c SaAmfNodeSwBundle \
    -a saAmfNodeSwBundlePathPrefix=/opt/demo \
    safInstalledSwBundle=safSmfBundle=demo,safAmfNode=SC-2,safAmfCluster=myAmfCluster
  
```


# 4. Unlock SUs.
## 4.1. SC-1.
```sh
  
$ amf-adm unlock-in safSu=1,safSg=demo,safApp=demo

$ amf-adm unlock safSu=1,safSg=demo,safApp=demo
  
```

## 4.2. SC-2.
```sh
  
$ amf-adm unlock-in safSu=2,safSg=demo,safApp=demo

$ amf-adm unlock safSu=2,safSg=demo,safApp=demo
  
```


# References
- https://opensaf.sourceforge.io/documentation.html
- SAI-AIS-AMF-B.04.01.AL, 8.3 DN Formats for AMF UML Classes