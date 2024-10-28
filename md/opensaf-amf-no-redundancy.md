<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    mermaid.initialize({
        look: 'handDrawn',
        theme: 'neutral',
    });
</script>


---
title:  'OpenSAF AMF - High Available SNMP'
---


# Create Daemon Program
Create directory `/opt/demo`.
```sh
mkdir /opt/demo
```

Create file `/opt/demo/loop.sh`.
```sh

counter=1

while true; do
    logger "hello $counter"
    ((counter++))
    sleep 5
done

```

Create file `/opt/demo/main.sh`.
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


# Create AMF BaseTypes
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


# Create AMF Types.
```sh

# swbundle
immcfg -c SaSmfSwBundle safSmfBundle=demo

# csi
immcfg -c SaAmfCSType safVersion=1,safCSType=demo

# comp
immcfg -c SaAmfCompType \
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
immcfg -c SaAmfSvcType safVersion=1,safSvcType=demo

# su
immcfg -c SaAmfSUType \
    -a saAmfSutIsExternal=0 \
    -a saAmfSutDefSUFailover=1 \
    -a saAmfSutProvidesSvcTypes=safVersion=1,safSvcType=demo \
    safVersion=1,safSuType=demo

# sg
immcfg -c SaAmfSGType \
    -a saAmfSgtRedundancyModel=1 \
    -a saAmfSgtValidSuTypes=safVersion=1,safSuType=demo \
    -a saAmfSgtDefAutoAdjustProb=10000000000 \
    -a saAmfSgtDefCompRestartProb=4000000000 \
    -a saAmfSgtDefCompRestartMax=10 \
    -a saAmfSgtDefSuRestartProb=4000000000 \
    -a saAmfSgtDefSuRestartMax=10 \
    safVersion=1,safSgType=demo

# app
immcfg -c SaAmfAppType \
    -a saAmfApptSGTypes=safVersion=1,safSgType=demo \
    safVersion=1,safAppType=demo

```


# Create AMF Connector Types
```sh

# connect si - csi
immcfg -c SaAmfSvcTypeCSTypes \
    'safMemberCSType=safVersion=1\,safCSType=demo,safVersion=1,safSvcType=demo'

# connect comp - csi
immcfg -c SaAmfCtCsType \
    -a saAmfCtCompCapability=1 \
    'safSupportedCsType=safVersion=1\,safCSType=demo,safVersion=1,safCompType=demo'

# connect su - comp
immcfg -c SaAmfSutCompType \
    'safMemberCompType=safVersion=1\,safCompType=demo,safVersion=1,safSuType=demo'

```

# Create Common Entities.
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
    safSi=1,safApp=demo

immcfg -c SaAmfCSI \
    -a saAmfCSType=safVersion=1,safCSType=demo \
    safCsi=demo,safSi=1,safApp=demo

```

# 3.4. Create Entities for SC-1.

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

$ immcfg -c SaAmfNodeSwBundle \
    -a saAmfNodeSwBundlePathPrefix=/opt/demo \
    safInstalledSwBundle=safSmfBundle=demo,safAmfNode=SC-1,safAmfCluster=myAmfCluster

```

# Create Connector Entities.
```sh

$ immcfg -c SaAmfCompCsType \
    'safSupportedCsType=safVersion=1\,safCSType=demo,safComp=demo,safSu=1,safSg=demo,safApp=demo'

```

# Unlock SU on SC-1.
```sh
$ amf-adm unlock-in safSu=1,safSg=demo,safApp=demo

$ amf-adm unlock safSu=1,safSg=demo,safApp=demo

```