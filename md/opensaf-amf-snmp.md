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
Setup a cluster with 2 SCs.


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
    -a saAmfCtRelPathAmStartCmd='./../usr/local/sbin/amfpm --start' \
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

# References
- https://opensaf.sourceforge.io/documentation.html
- SAI-AIS-AMF-B.04.01.AL, 8.3 DN Formats for Objects