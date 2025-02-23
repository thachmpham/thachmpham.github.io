---
title:  'OpenSAF SMF'
---


# 1. Introduction
Software Management Framwork (SMF) supports high availability during system upgrade, such as addition, removal, replacement or reconfiguration of software or hardware elements.

CLI commands:

- smf-adm
- smf-find
- smf-state

# 2. Labs
In this lab, we will create a demo campaign and install to the cluster.  

- Create a file `/opt/demo/demo_campaign.xml`.

```xml
  
<?xml version="1.0" encoding="utf-8"?>

<upgradeCampaign safSmfCampaign="safSmfCampaign=demo">

    <campaignInitialization>
        <addToImm>
            <softwareBundle name="safSmfBundle=demo">
                <removal>
                    <offline command="logger" args="demo: offline remove smf bundle" saSmfBundleRemoveOfflineScope="0"/>
                    <online  command="logger" args="demo: online  remove smf bundle"/>
                </removal>
                <installation>
                    <offline command="logger" args="demo: offline install smf bundle" saSmfBundleInstallOfflineScope="0"/>
                    <online  command="logger" args="demo: online  install smf bundle"/>
                </installation>
                <defaultCliTimeout/>
            </softwareBundle>
        </addToImm>
    </campaignInitialization>

    <upgradeProcedure safSmfProcedure="safSmfProc=SingleStepProc">
        <upgradeMethod>
            <singleStepUpgrade>
                <upgradeScope>
                    <forAddRemove>
                        <deactivationUnit/>
                            <swAdd bundleDN="safSmfBundle=demo" pathnamePrefix="/opt/demo">
                                <plmExecEnv amfNode="safAmfNode=SC-1,safAmfCluster=myAmfCluster"/>
                            </swAdd>
                        <activationUnit>
                            <swAdd bundleDN="safSmfBundle=demo" pathnamePrefix="/opt/demo">
                                <plmExecEnv amfNode="safAmfNode=SC-1,safAmfCluster=myAmfCluster"/>
                            </swAdd>
                        </activationUnit>
                    </forAddRemove>
                </upgradeScope>
            </singleStepUpgrade>
        </upgradeMethod>
    </upgradeProcedure>

    <campaignWrapup>
       <campCompleteAction>
			<doCliCommand   command="logger" args="demo: do   complete action"></doCliCommand>
			<undoCliCommand command="logger" args="demo: undo complete action"></undoCliCommand>
			<plmExecEnv amfNode="safAmfNode=SC-1,safAmfCluster=myAmfCluster"></plmExecEnv>
		</campCompleteAction>
    </campaignWrapup>

</upgradeCampaign>
  
```

- Create SaSmfCampaign object.
```sh
  
$ immcfg -c SaSmfCampaign -a saSmfCmpgFileUri=/opt/demo/demo_campaign.xml safSmfCampaign=demo,safApp=safSmfService
  
```

- Execute campaign.
```sh
  
$ smf-adm execute safSmfCampaign=demo,safApp=safSmfService
  
```
```sh
  
# syslog
osafsmfd: NO STEP: Executing AU lock step safSmfStep=0001,safSmfProc=SingleStepProc,safSmfCampaign=demo,safApp=safSmfService

osafsmfd: NO STEP: Online installation of new software
osafsmfnd: NO Successful start of command execution: logger demo: online  install smf bundle, timeout 600000
root: demo: online install smf bundle

osafsmfd: NO STEP: Lock deactivation units
osafsmfd: NO STEP: Terminate deactivation units

osafsmfd: NO STEP: Offline uninstallation of old software

osafsmfd: NO STEP: Modify information model and set maintenance status

osafsmfd: NO STEP: Offline installation of new software
osafsmfnd: NO Successful start of command execution: logger demo: offline install smf bundle, timeout 600000
root: demo: offline install smf bundle
osafsmfd: NO STEP: Create new SaAmfNodeSwBundle objects

osafsmfd: NO STEP: Instantiate activation units
osafsmfd: NO STEP: Unlock activation units

osafsmfd: NO STEP: Upgrade AU lock step completed safSmfStep=0001,safSmfProc=SingleStepProc,safSmfCampaign=demo,safApp=safSmfService

osafsmfd: NO PROC: Online uninstallation of old software
osafsmfd: NO PROC: Delete SaAmfNodeSwBundle objects

osafsmfd: NO PROC: All steps has been executed
osafsmfd: NO PROC: Start procedure wrapup actions

osafsmfnd: NO Successful start of command execution: logger demo: do   complete action, timeout 600000
root: demo: do complete action
  
```

- Commit.
```sh
  
$ smf-adm commit safSmfCampaign=demo,safApp=safSmfService
  
```
```sh
   
# syslog
osafsmfd: NO CAMP: Commit upgrade campaign safSmfCampaign=demo
osafsmfd: NO CAMP: Campaign wrapup, start wrapup actions (0)
osafsmfd: NO CAMP: Campaign wrapup, start remove from IMM (0)
osafsmfd: NO CAMP: Campaign wrapup actions completed
osafsmfd: NO CAMP: Campaign wrapup, reset saAmfSUMaintenanceCampaign flags
osafsmfd: NO CAMP: Campaign wrapup, Remove runtime objects
osafsmfd: NO CAMP: Campaign wrapup, Remove config objects
osafsmfd: NO CAMP: Upgrade campaign committed safSmfCampaign=demo
   
```

- Check result.
```sh
  
$ amf-find nodeswbundle
  
```
```sh
  
safInstalledSwBundle=safSmfBundle=OpenSAF,safAmfNode=SC-1,safAmfCluster=myAmfCluster
safInstalledSwBundle=safSmfBundle=demo,safAmfNode=SC-1,safAmfCluster=myAmfCluster
  
```

# 3. Troubleshooting
## 3.1. Environment Variables
- Set environment variables in `/etc/opensaf/smfd.conf`.
```sh
  
# needed for script smf-repository-check
export REPOSITORY=/root/smf

# needed for script smf-backup-create
export SMFREPOSITORY=/root/smf
export SMF_BACKUP_DIR=/root/smf
  
```

- Restart opensafd to load new environment vairables.
```sh
  
$ /etc/init.d/opensafd restart
  
```


# References
- https://opensaf.sourceforge.io/documentation.html
- SAI-AIS-SMF-A.01.02, Figure 3: Upgrade Campaign Activity Diagram
- SAI-AIS-SMF-A.01.02, 3.3.2.3 Actions of The Upgrade Step