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

- Commit.
```sh
  
$ smf-adm commit safSmfCampaign=demo,safApp=safSmfService
  
```

# 3. Troubleshooting
## 3.1. Setup environment variables
Set environment variables in `/etc/opensaf/smfd.conf`.
```sh
  
# needed for script smf-repository-check
export REPOSITORY=/root/smf

# needed for script smf-backup-create
export SMFREPOSITORY=/root/smf
export SMF_BACKUP_DIR=/root/smf
  
```

Restart opensafd to load new environment vairables.
```sh
  
$ /etc/init.d/opensafd restart
  
```


# References
- https://opensaf.sourceforge.io/documentation.html