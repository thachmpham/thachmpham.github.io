---
title:  'OpenSAF AMF - High Available SNMP'
---


<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    mermaid.initialize({
        look: 'handDrawn',
        theme: 'neutral',
    });
</script>


# 1. Introduction.

# 2. Setup Cluster.
- Setup a cluster with 2 SCs.
- Install snmp.
```sh

$ apt install -y snmpd snmp

```

# 3. Setup IMM Model.
## 3.1. Create BaseTypes.
```sh
# app
$ immcfg -c SaAmfAppBaseType safAppType=net-snmp

# sg
$ immcfg -c SaAmfSGBaseType safSgType=2N-net-snmp

# su
$ immcfg -c SaAmfSUBaseType safSuType=snmpd

# si
$ immcfg -c SaAmfSvcBaseType safSvcType=snmpd

# comp
$ immcfg -c SaAmfCompBaseType safCompType=snmpd

# csi
$ immcfg -c SaAmfCSBaseType safCSType=snmpd

```

## 3.2. Create Types.
Dependencies between entity types.

<pre class="mermaid">
classDiagram
    class SaAmfAppType {
        saAmfApptSGTypes
    }
    SaAmfAppType --> SaAmfSGType


    class SaAmfSGType {
        saAmfSgtValidSuTypes
    }
    SaAmfSGType --> SaAmfSUType


    class SaAmfSUType {
        saAmfSutProvidesSvcTypes
    }
    SaAmfSUType --> SaAmfSvcType


    class SaAmfCompType {
        saAmfCtSwBundle
    }
    SaAmfCompType --> SaSmfSwBundle

    class SaAmfCSType
</pre>


```sh

# swbundle
immcfg -c SaSmfSwBundle safSmfBundle=net-snmp-5.6.1-4.5.1.x86_64

# csi
immcfg -c SaAmfCSType safVersion=1,safCSType=snmpd

# comp
immcfg -c SaAmfCompType \
    -a saAmfCtCompCategory=8 \
    -a saAmfCtSwBundle=safSmfBundle=net-snmp-5.6.1-4.5.1.x86_64 \
    -a saAmfCtDefClcCliTimeout=10000000000 \
    -a saAmfCtRelPathInstantiateCmd='snmpd start' \
    -a saAmfCtRelPathCleanupCmd='snmpd stop' \
    -a saAmfCtRelPathTerminateCmd='snmpd stop' \
    -a saAmfCtRelPathAmStartCmd='../../usr/local/sbin/amfpm --start' \
    -a saAmfCtRelPathAmStopCmd='../../usr/local/sbin/amfpm --stop' \
    -a saAmfCtDefRecoveryOnError=3 \
    -a saAmfCtDefDisableRestart=0 \
    safVersion=5.6.1-4.5.1,safCompType=snmpd

# si
immcfg -c SaAmfSvcType safVersion=1,safSvcType=snmpd

# su
immcfg -c SaAmfSUType \
    -a saAmfSutIsExternal=0 \
    -a saAmfSutDefSUFailover=1 \
    -a saAmfSutProvidesSvcTypes=safVersion=1,safSvcType=snmpd \
    safVersion=1,safSuType=snmpd

# sg
immcfg -c SaAmfSGType \
    -a saAmfSgtRedundancyModel=1 \
    -a saAmfSgtValidSuTypes=safVersion=1,safSuType=snmpd \
    -a saAmfSgtDefAutoAdjustProb=10000000000 \
    -a saAmfSgtDefCompRestartProb=4000000000 \
    -a saAmfSgtDefCompRestartMax=10 \
    -a saAmfSgtDefSuRestartProb=4000000000 \
    -a saAmfSgtDefSuRestartMax=10 \
    safVersion=1,safSgType=2N-net-snmp

# app
immcfg -c SaAmfAppType \
    -a saAmfApptSGTypes=safVersion=1,safSgType=2N-net-snmp \
    safVersion=1,safAppType=net-snmp

```


## 2.3. Create Connector Types.
<pre class="mermaid">
classDiagram
    class SaAmfSvcTypeCSTypes {
        dn = csType + svcType
    }
    SaAmfSvcTypeCSTypes --> SaAmfSvcType
    SaAmfSvcTypeCSTypes --> SaAmfCSType

    class SaAmfCtCsType {
        dn = csType + compType
    }
    SaAmfCtCsType --> SaAmfCompType
    SaAmfCtCsType --> SaAmfCSType

    class SaAmfSutCompType {
        dn = compType + suType
    }
    SaAmfSutCompType --> SaAmfSUType
    SaAmfSutCompType --> SaAmfCompType
</pre>

```sh

# connect si - csi
immcfg -c SaAmfSvcTypeCSTypes \
    'safMemberCSType=safVersion=1\,safCSType=snmpd,safVersion=1,safSvcType=snmpd'

# connect comp - csi
immcfg -c SaAmfCtCsType \
    -a saAmfCtCompCapability=1 \
    'safSupportedCsType=safVersion=1\,safCSType=snmpd,safVersion=5.6.1-4.5.1,safCompType=snmpd'

# connect su - comp
immcfg -c SaAmfSutCompType \
    'safMemberCompType=safVersion=5.6.1-4.5.1\,safCompType=snmpd,safVersion=1,safSuType=snmpd'

```


## 3.3. Create Common Entities.
<pre class="mermaid">
classDiagram
    class SaAmfApplication {
        dn: "safApp=..."
    }

    class SaAmfSG {
        dn: "safSg=...,safApp=..."
    }
    SaAmfSG --> SaAmfApplication

    class SaAmfSI {
        dn: "safSi=...,safApp=..."
        saAmfSIProtectedbySG
    }
    SaAmfSI --> SaAmfApplication
    SaAmfSI --> SaAmfSG

    class SaAmfCSI {
        dn: "safCsi=...,safSi=...,safApp=..."
    }
    SaAmfCSI --> SaAmfSI

</pre>


```sh

immcfg -c SaAmfApplication \
    -a saAmfAppType=safVersion=1,safAppType=net-snmp \
    safApp=net-snmp

immcfg -c SaAmfSG \
    -a saAmfSGType=safVersion=1,safSgType=2N-net-snmp \
    -a saAmfSGAutoRepair=0 \
    -a saAmfSGAutoAdjust=0 \
    -a saAmfSGNumPrefInserviceSUs=10 \
    -a saAmfSGNumPrefAssignedSUs=10 \
    safSg=2N,safApp=net-snmp

immcfg -c SaAmfSI \
    -a saAmfSvcType=safVersion=1,safSvcType=snmpd \
    -a saAmfSIProtectedbySG=safSg=2N,safApp=net-snmp \
    -a saAmfSIRank=1 \
    safSi=1,safApp=net-snmp

immcfg -c SaAmfCSI \
    -a saAmfCSType=safVersion=1,safCSType=snmpd \
    safCsi=snmpd,safSi=1,safApp=net-snmp

```


## 3.4. Create Entities for SC-1.
### 3.4.1. Create Entities.
<pre class="mermaid">
classDiagram
    class SaAmfSU {
        dn: "safSu=...,safSg=..."
        safAmfSUHostNodeOrNodeGroup
    }
    SaAmfSU --> SaAmfNode
    SaAmfSU --> SaAmfNodeGroup

    class SaAmfComp {
        dn: "safComp=...,safSu=..."
        saAmfCtRelPathInstantiateCmd
        saAmfCtRelPathAmStartCmd
    }
    SaAmfComp --> SaAmfSU
    SaAmfComp --> SaAmfNodeSwBundle

    class SaAmfNodeSwBundle {
        dn: "safInstalledSwBundle=...,safAmfNode=..."
        saAmfNodeSwBundlePathPrefix
    }
    SaAmfNodeSwBundle --> SaAmfNode

    class SaAmfNode {
        dn: "safAmfNode=...,safAmfCluster=..."
    }

    class SaAmfNodeGroup {
        dn: "safAmfNodeGroup=...,safAmfCluster=..."
        saAmfNGNodeList
    }
    SaAmfNodeGroup --> SaAmfNode
</pre>

```sh
    
$ immcfg -c SaAmfSU \
    -a saAmfSUType=safVersion=1,safSuType=snmpd \
    -a saAmfSUHostNodeOrNodeGroup=safAmfNode=SC-1,safAmfCluster=myAmfCluster \
    -a saAmfSURank=1 \
    -a saAmfSUAdminState=3 \
    safSu=1,safSg=2N,safApp=net-snmp
  
$ immcfg -c SaAmfComp \
    -a saAmfCompType=safVersion=5.6.1-4.5.1,safCompType=snmpd \
    safComp=snmpd,safSu=1,safSg=2N,safApp=net-snmp

$ immcfg -c SaAmfNodeSwBundle \
    -a saAmfNodeSwBundlePathPrefix=/etc/init.d \
    safInstalledSwBundle=safSmfBundle=net-snmp-5.6.1-4.5.1.x86_64,safAmfNode=SC-1,safAmfCluster=myAmfCluster

```

### 3.4.2. Create Connector Entities.
<pre class="mermaid">
classDiagram
    class SaAmfCompCsType {
        dn: "safSupportedCsType=...,safCSType=...,safComp=...,safSu=...,safSg=...,safApp=..."
    }
    SaAmfCompCsType --> SaAmfComp
    SaAmfCompCsType --> SaAmfCSType

    SaAmfCSType <|-- SaAmfCSI
</pre>

```sh

$ immcfg -c SaAmfCompCsType \
    'safSupportedCsType=safVersion=1\,safCSType=snmpd,safComp=snmpd,safSu=1,safSg=2N,safApp=net-snmp'

```

## 3.4.3. Unlock SU on SC-1.
```sh
$ amf-adm unlock-in safSu=1,safSg=2N,safApp=net-snmp

$ amf-adm unlock safSu=1,safSg=2N,safApp=net-snmp

```

Check su status.
```sh
$ amf-state su
safSu=1,safSg=2N,safApp=net-snmp
    saAmfSUAdminState=UNLOCKED(1)
    saAmfSUOperState=ENABLED(1)
    saAmfSUPresenceState=INSTANTIATED(3)
    saAmfSUReadinessState=IN-SERVICE(2)
```

Check `/var/log/syslog`.
```sh
SC-1 osafamfnd[352]: NO Assigning 'safSi=1,safApp=net-snmp' ACTIVE to 'safSu=1,safSg=2N,safApp=net-snmp'
SC-1 osafamfnd[352]: NO 'safSu=1,safSg=2N,safApp=net-snmp' Presence State UNINSTANTIATED => INSTANTIATING
SC-1 osafamfnd[352]: NO 'safSu=1,safSg=2N,safApp=net-snmp' Presence State INSTANTIATING => INSTANTIATED
SC-1 osafamfnd[352]: NO Assigned 'safSi=1,safApp=net-snmp' ACTIVE to 'safSu=1,safSg=2N,safApp=net-snmp'
SC-1 snmpd[1185]: NET-SNMP version 5.7.3
```

Check snmp status.
```sh
  
$ /etc/init.d/snmpd status
snmpd is running
  
```

# References
- https://opensaf.sourceforge.io/documentation.html
- SAI-AIS-AMF-B.04.01.AL, 8.3 DN Formats for AMF UML Classes