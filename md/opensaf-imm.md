---
title:  'OpenSAF IMM'
---


# 1. Introduction.
In OpenSAF, IMM (Information Model Management) is responsible for managing and storing the system's configuration and status

IMM works like a central database where all services and components update their information. It keeps data consistent across all nodes, so when something changes like a service failure or configuration update, the whole system is aware and can react.

OpenSAF provides the following commands to interact with IMM tasks:  

- immdump
- immfind
- immlist
- immcfg
- immadm

# 2. Commands.
## 2.1. immdump.
The command immdump is used to dump or write the IMM model to a file.
```s
  
# dump imm model to terminal
$ immdump

# dump imm model to file
$ immdump output.xml
  
```


## 2.2. immfind.
The command immfind is used to find IMM objects.

Search for all objects
```sh

$ immfind
  
```

Search by class SaAmfNode
```sh
  
$ immfind -c SaAmfNode
safAmfNode=SC-1,safAmfCluster=myAmfCluster
  
```

## 2.3. immlist.
The command immlist is used to print list of attributes of an IMM object.

```sh
  
$ immlist safAmfNode=SC-1,safAmfCluster=myAmfCluster
Name                                               Type         Value(s)
========================================================================
safAmfNode                                         SA_STRING_T  safAmfNode=SC-1 
saAmfNodeSuFailoverMax                             SA_UINT32_T  2 (0x2)
saAmfNodeSuFailOverProb                            SA_TIME_T    1200000000000
saAmfNodeOperState                                 SA_UINT32_T  1 (0x1)
saAmfNodeFailfastOnTerminationFailure              SA_UINT32_T  0 (0x0)
saAmfNodeFailfastOnInstantiationFailure            SA_UINT32_T  0 (0x0)
saAmfNodeClmNode                                   SA_NAME_T    safNode=SC-1,safCluster=myClmCluster (36) 
saAmfNodeCapacity                                  SA_STRING_T  <Empty>
saAmfNodeAutoRepair                                SA_UINT32_T  1 (0x1)
saAmfNodeAdminState                                SA_UINT32_T  1 (0x1)
SaImmAttrImplementerName                           SA_STRING_T  safAmfService 
SaImmAttrClassName                                 SA_STRING_T  SaAmfNode 
SaImmAttrAdminOwnerName                            SA_STRING_T  IMMLOADER
  
```


## 2.4. immcfg.
The command immcfg is used to change configuration attribute of an IMM object.

Create an object.
```sh
  
$ immcfg -c SaAmfApplication -a saAmfAppType=safVersion=4.0.0,safAppType=OpenSafApplicationType safApp=myTestApp1
  
```

Delete an object
```sh
  
$ immcfg -d safApp=myTestApp1
  
```

Change one attribute of an object.
```sh
  
$ immcfg -a saAmfNodeSuFailoverMax=3 safAmfNode=SC-1,safAmfCluster=myAmfCluster
  
```

<br>

Import an IMM XML file.

- Create a file named `employee.xml`.
```xml
  
<?xml version="1.0" encoding="utf-8"?>
<imm:IMM-contents xmlns:imm="http://www.saforum.org/IMMSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="SAI-AIS-IMM-XSD-A.01.01.xsd">
    
    <class name="Employee">
        <category>SA_CONFIG</category>
        <rdn>
            <name>employeeRdn</name>
            <type>SA_STRING_T</type>
            <category>SA_CONFIG</category>
            <flag>SA_INITIALIZED</flag>
        </rdn>
        <attr>
            <name>employeeName</name>
            <type>SA_STRING_T</type>
            <category>SA_CONFIG</category>
            <flag>SA_WRITABLE</flag>
        </attr>
        <attr>
            <name>employeeAge</name>
            <type>SA_UINT32_T</type>
            <category>SA_CONFIG</category>
            <flag>SA_WRITABLE</flag>
        </attr>               
    </class>

    <object class="Employee">
        <dn>employee=Alex</dn>
        <attr>
            <name>employeeName</name>
            <value>Alex</value>
        </attr>
        <attr>
            <name>employeeAge</name>
            <value>32</value>
        </attr>
    </object>

</imm:IMM-contents>
  
```

- Import and check result.
```sh
  
$ immcfg -f employee.xml
  
```
```sh
  
$ immfind -c Employee
employee=Alex

$ immlist employee=Alex
Name                                               Type         Value(s)
========================================================================
employeeRdn                                        SA_STRING_T  employee=Alex 
employeeName                                       SA_STRING_T  Alex 
employeeAge                                        SA_UINT32_T  32 (0x20)
SaImmAttrImplementerName                           SA_STRING_T  <Empty>
SaImmAttrClassName                                 SA_STRING_T  Employee 
SaImmAttrAdminOwnerName                            SA_STRING_T  <Empty>
  
```

## 2.5. immadm.
The command immcfg is used to change configuration attribute of an IMM object.

# References
- https://opensaf.sourceforge.io/documentation.html

