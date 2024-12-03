---
title:  'OpenSAF AMF'
subtitle: 'OpenSAF SA Aware Sample App'
---


# Introduction
#### Agenda.
- Build sample app opensaf/samples/amf/sa_aware.
- Install sample app to SC-1, SC-2.

#### Prerequisite.
- [Setup a cluster with 2 SCs.](./opensaf-2sc.html)


# Setup Sample App on SC-1.
Generate Makefile.
```sh
  
SC-1$ cd /root/opensaf-staging/samples
SC-1$ ./bootstrap.sh
SC-1$ ./configure
  
```

Build, install sample app. It will be installed to /opt/amf_demo
```sh
  
SC-1$ cd /root/opensaf-staging/samples/amf/sa_aware
SC-1$ make
SC-1$ make install
  
```

Import IMM configure.
```sh
  
SC-1$ immcfg -f /root/opensaf-staging/samples/amf/sa_aware/AppConfig-2N.xml
  
```

Check status.
```sh
  
SC-1$ amf-find su
safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1
safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1

SC-1$ amf-state su all safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1
safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1
        saAmfSUAdminState=LOCKED-INSTANTIATION(3)
        saAmfSUOperState=ENABLED(1)
        saAmfSUPresenceState=UNINSTANTIATED(1)
        saAmfSUReadinessState=OUT-OF-SERVICE(1)
  
```

Instantiate the sample app.
```sh
SC-1$ amf-adm unlock-in safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1  
```

Syslog
```log
  
SC-1 osafamfnd[388]: NO 'safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1' Presence State UNINSTANTIATED => INSTANTIATING
SC-1 amf_demo[15182]: 'safComp=AmfDemo,safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1' started
SC-1 amf_demo[15182]: before saAmfComponentRegister [safComp=AmfDemo,safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1]
SC-1 osafamfnd[388]: NO 'safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1' Presence State INSTANTIATING => INSTANTIATED
SC-1 amf_demo[15182]: after saAmfComponentRegister 
SC-1 amf_demo[15182]: Registered with AMF and HC started
SC-1 amf_demo[15182]: Health check 1
  
```

Call flow
```sh
  
amf-adm unlock-in
    --> SaAmfCompType.saAmfCtRelPathInstantiateCmd+saAmfCtDefInstantiateCmdArgv
    --> /opt/amf_demo/amf_demo_script instantiate
    --> /opt/amf_demo/amf_demo
    --> main()
  
```

Unlock the sample app.
```sh
  
SC-1$ amf-adm unlock safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1
  
```

Syslog
```log
  
SC-1 amf_demo[15182]: Health check 56
SC-1 osafamfnd[388]: NO Assigning 'safSi=AmfDemo,safApp=AmfDemo1' ACTIVE to 'safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1'
SC-1 amf_demo[15182]: CSI Set - add 'safCsi=AmfDemo,safSi=AmfDemo,safApp=AmfDemo1' HAState Active
SC-1 amf_demo[15182]:     name: AmfDemo1, value: AAAA
SC-1 amf_demo[15182]:     name: AmfDemo1, value: BBBB
SC-1 osafamfnd[388]: NO Assigned 'safSi=AmfDemo,safApp=AmfDemo1' ACTIVE to 'safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1'
SC-1 amf_demo[15182]: Health check 57
SC-1 amf_demo[15182]: Health check 58
  
```

Call flow
```sh
   
amf-adm unlock
    --> amf_csi_set_callback()
        --> saAmfResponse_4()
  
```

# Build & Install Sample App on SC-2.
Generate Makefile.
```sh
  
SC-2$ cd /root/opensaf-staging/samples
SC-2$ ./bootstrap.sh
SC-2$ ./configure
    
```

Build, install sample app. It will be installed to /opt/amf_demo
```sh
  
SC-2$ cd /root/opensaf-staging/samples/amf/sa_aware
SC-2$ make
SC-2$ make install
  
```

Check status.
```sh
  
SC-2$ amf-find su
safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1
safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1

SC-2$ amf-state su all safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1
safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1
        saAmfSUAdminState=LOCKED-INSTANTIATION(3)
        saAmfSUOperState=ENABLED(1)
        saAmfSUPresenceState=UNINSTANTIATED(1)
        saAmfSUReadinessState=OUT-OF-SERVICE(1)
  
```

Instantiate the sample app.
```sh
SC-2$ amf-adm unlock-in safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1  
```


Syslog
```log
  
SC-2 osafamfnd[164]: NO 'safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1' Presence State UNINSTANTIATED => INSTANTIATING
SC-2 amf_demo[2893]: 'safComp=AmfDemo,safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1' started
SC-2 amf_demo[2893]: before saAmfComponentRegister [safComp=AmfDemo,safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1]
SC-2 osafamfnd[164]: NO 'safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1' Presence State INSTANTIATING => INSTANTIATED
SC-2 amf_demo[2893]: after saAmfComponentRegister 
SC-2 amf_demo[2893]: Registered with AMF and HC started
SC-2 amf_demo[2893]: Health check 1
  
```

Call flow
```sh
  
amf-adm unlock-in
    --> SaAmfCompType.saAmfCtRelPathInstantiateCmd+saAmfCtDefInstantiateCmdArgv
    --> /opt/amf_demo/amf_demo_script instantiate
    --> /opt/amf_demo/amf_demo
    --> main()
  
```

Unlock the sample app.
```sh
  
SC-2$ amf-adm unlock safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1
  
```

Syslog
```log
  
osafamfnd[164]: NO Assigning 'safSi=AmfDemo,safApp=AmfDemo1' STANDBY to 'safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1'
amf_demo[2893]: CSI Set - add 'safCsi=AmfDemo,safSi=AmfDemo,safApp=AmfDemo1' HAState Standby
amf_demo[2893]:     name: AmfDemo1, value: AAAA
amf_demo[2893]:     name: AmfDemo1, value: BBBB
osafamfnd[164]: NO Assigned 'safSi=AmfDemo,safApp=AmfDemo1' STANDBY to 'safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1'
  
```


Call flow
```sh
  
amf-adm unlock
    --> amf_csi_set_callback()
        --> saAmfResponse_4()
  
```


# Fault Tolerance inside a Node.
```sh
  
SC-1$ kill `pidof amf_demo`
  
```

Syslog.
```log
  
SC-1 amf_demo[16111]: Health check 27
SC-1 amf_demo[16111]: exiting (caught term signal)
SC-1 osafamfnd[15910]: NO 'safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1' component restart probation timer started (timeout: 4000000000 ns)
SC-1 osafamfnd[15910]: NO Restarting a component of 'safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1' (comp restart count: 1)
SC-1 osafamfnd[15910]: NO 'safComp=AmfDemo,safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1' faulted due to 'avaDown' : Recovery is 'componentRestart'
SC-1 osafamfnd[15910]: NO 'safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1' Presence State INSTANTIATED => RESTARTING
SC-1 amf_demo[16163]: 'safComp=AmfDemo,safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1' started
SC-1 amf_demo[16163]: before saAmfComponentRegister [safComp=AmfDemo,safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1]
SC-1 osafamfnd[15910]: NO 'safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1' Presence State RESTARTING => INSTANTIATED
SC-1 amf_demo[16163]: after saAmfComponentRegister 
SC-1 amf_demo[16163]: Registered with AMF and HC started
SC-1 amf_demo[16163]: CSI Set - add 'safCsi=AmfDemo,safSi=AmfDemo,safApp=AmfDemo1' HAState Active
SC-1 amf_demo[16163]:     name: AmfDemo1, value: AAAA
SC-1 amf_demo[16163]:     name: AmfDemo1, value: BBBB
SC-1 amf_demo[16163]: Health check 1
  
```

Call flow.
```sh
  
kill `pidof amf_demo`
    --> osafamfnd component restart
    --> /opt/amf_demo/amf_demo
    --> main()
    --> amf_csi_set_callback()
        --> saAmfResponse_4()
  
```


# Fault Tolerance in Two Nodes.
```sh

SC-1$ amf-adm lock safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1
SC-1$ amf-adm lock-in safSu=SU1,safSg=AmfDemo,safApp=AmfDemo1

```


Syslog on SC-2
```log
  
SC-2 osafamfnd[3820]: NO Assigning 'safSi=AmfDemo,safApp=AmfDemo1' ACTIVE to 'safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1'
SC-2 amf_demo[4021]: CSI Set - HAState Active for all assigned CSIs
SC-2 osafamfnd[3820]: NO Assigned 'safSi=AmfDemo,safApp=AmfDemo1' ACTIVE to 'safSu=SU2,safSg=AmfDemo,safApp=AmfDemo1'  
  
```


# References
- https://opensaf.sourceforge.io/documentation.html