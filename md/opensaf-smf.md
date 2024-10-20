---
title:  'OpenSAF SMF'
---


# 1. Introduction.
Software Management Framwork (SMF) supports high availability during system upgrade, such as addition, removal, replacement or reconfiguration of software or hardware elements.

CLI commands:

- smf-adm
- smf-find
- smf-state

Admin Operations:
```c++
  
typedef enum
{
    SA_SMF_ADMIN_EXECUTE    = 1,
    SA_SMF_ADMIN_ROLLBACK   = 2,
    SA_SMF_ADMIN_SUSPEND    = 3,
    SA_SMF_ADMIN_COMMIT     = 4,
}
SaSmfAdminOperationIdT;
  
```

# 2. Labs.
## 2.1. Install Campaign on a Node.
### 2.1.1. Steps.
Create a file name `test_campaign.xml`.
```sh
  
$ mkdir -p /root/smf
$ touch /root/smf/test_campaign.xml
  
```

Copy the below text to `test_campaign.xml`.
```xml
  
<?xml version="1.0" encoding="utf-8"?>

<upgradeCampaign xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" safSmfCampaign="safSmfCampaign=install_bundles">

    <campaignInitialization>
        <addToImm>
            <softwareBundle name="safSmfBundle=TestCampaign">
                <removal>
                    <offline command="logger" args="-t TestCampaign removal offline" saSmfBundleRemoveOfflineScope="0"/>
                    <online command="logger" args="-t TestCampaign removal online"/>
                </removal>
                <installation>
                    <offline command="logger" args="-t TestCampaign installation offline" saSmfBundleInstallOfflineScope="0"/>
                    <online command="logger" args="-t TestCampaign installation online"/>
                </installation>
            </softwareBundle>
        </addToImm>
    </campaignInitialization>

    <upgradeProcedure safSmfProcedure="safSmfProc=SingleStepProc">
        <upgradeMethod>
            <singleStepUpgrade>
                <upgradeScope>
                    <forAddRemove>
                        <deactivationUnit/>
                        <activationUnit>
                            <swAdd bundleDN="safSmfBundle=TestCampaign" pathnamePrefix="/root/smf/test_campaign">
                                <plmExecEnv amfNode="safAmfNode=SC-1,safAmfCluster=myAmfCluster"/>
                            </swAdd>
                        </activationUnit>
                    </forAddRemove>
                </upgradeScope>
            </singleStepUpgrade>
        </upgradeMethod>
    </upgradeProcedure>

</upgradeCampaign>
  
```

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

Install campaign.
```sh
  
$ immcfg -c SaSmfCampaign -a saSmfCmpgFileUri=/root/smf/test_campaign.xml \
    safSmfCampaign=TestCampaign,safApp=safSmfService

$ immadm -o 1 safSmfCampaign=TestCampaign,safApp=safSmfService

$ immlist safSmfCampaign=TestCampaign,safApp=safSmfService

$ immadm -o 4 safSmfCampaign=TestCampaign,safApp=safSmfService
  
```


### 2.1.2. Expected Outcome.
The TestCampaign bundles should exist.
```sh

$ immfind -c SaSmfSwBundle
safSmfBundle=OpenSAF
safSmfBundle=TestCampaign

$ immfind -c SaAmfNodeSwBundle
safInstalledSwBundle=safSmfBundle=OpenSAF,safAmfNode=SC-1,safAmfCluster=myAmfCluster
safInstalledSwBundle=safSmfBundle=TestCampaign,safAmfNode=SC-1,safAmfCluster=myAmfCluster
  
```


# 3. Tips
## 3.1. Monitor syslog.
Before installing campaign, monitor syslog to check the progress.
```sh
  
$ /etc/init.d/rsyslog start
$ tail -f /var/log/syslog
  
```

# References
- https://opensaf.sourceforge.io/documentation.html