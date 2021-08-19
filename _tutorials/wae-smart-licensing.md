---
published: true
date: '2021-08-19 14:45 +0800'
title: WAN Automation Engine smart licensing
author: Fung Lim
tags:
  - WAN Automation Engine
  - Smart Licensing
excerpt: Cisco WAE Smart Licensing Primer
---
{% include toc %}

## WAN Automation Engine Smart Licensing

Smart Licensing provides for a flexible software licensing model that simplifies the way you activate and manage WAE licenses across your organization. Cisco WAE has been supporting Smart Licensing since 7.2 release. This document provides a primer on using Smart Licensing on Cisco WAE.

Before you proceed, ensure that
* You have a valid SMART Account with administration priviledges.
* Cisco WAE server has been installed and is in RUNNING state.

At this point, there are no licenses activated or installed on the WAE server.

Note: If you are migrating from traditional node based license, the **MATE_Dedicated.lic** license file will need to be removed from the current installation before proceeding.


### Check requisites

You may confirm the status of the current WAE server installation using the following commands

```
[wae@wae ~]$ supervisorctl status
wae:kafka                        RUNNING   pid 6519, uptime 0:06:04
wae:logrotate                    RUNNING   pid 6518, uptime 0:06:04
wae:wae-monitor                  RUNNING   pid 6517, uptime 0:06:04
wae:waectl                       RUNNING   pid 6516, uptime 0:06:04
wae:zookeeper                    RUNNING   pid 6515, uptime 0:06:04
```

```
[wae@wae ~]$ license_check
License files searched:
/home/wae/.cariden/etc;/home/wae/wae7/etc;/home/wae/wae7/etc

There are no licenses available.

System HostID(s): 5254007e0f82
```


### Run license_install on WAE server

The next step is to run the **license_install** command on the WAE server. Replace **198.18.134.30** with the IP address of your WAE server and **password** with the WAE admin user password. A **MATE_Smart.lic** file will be created under the $WAE_HOME/.cariden/etc directory.

```
[wae@wae ~]$ source /home/wae/wae7/waerc
[wae@wae ~]$ license_install -smart-lic-host 198.18.134.30 -smart-lic-port 2022 -smart-lic-username admin -smart-lic-password password
License successfully installed.

[wae@wae ~]$ ls /home/wae/.cariden/etc
MATE_Smart.lic

[wae@wae ~]$ license_check
There are no licenses available.

WAE Server: "198.18.134.30"
```


### Generate registration token using Cisco Smart Software Manager

In order to do this, proceed to software.cisco.com and select **Smart Software Manager > Manage Licenses**. 

Under **General > Product Instance Registration Tokens**, select **New Token**. Enter the description, expiry date and desired number of users. After the registration token is generated, download the  file. Use a text editor to open and copy the **Token string** for use in the next step.



### Enable and register WAE server for Smart Licensing 

Login to WAE Web UI (https://<wae-ip-address>:8443/) using the admin user and password. On the WAE Web UI Dashboard, select Smart Licensing.
  
![WAE WebUI Smart Licensing Selection]({{site.baseurl}}/images/screenshot 2021-08-19 14.19.34.png)
  
Select Enable Smart Licensing
  
![Enable Smart Licensing Selection]({{site.baseurl}}/images/screenshot 2021-08-19 14.20.24.png)
  
Select Register in order to register WAE with Cisco Smart Software Licensing.
  

![Smart Software License Registration]({{site.baseurl}}/images/screenshot 2021-08-19 21.51.31.png)

  
Enter the Token string into the Product Instance Registration Token text box and select Register. If the registration is successful, you will be prompted with the message Registration completed successfully.

### Select desired licenses 
  
You may now select the desired licenses along with the node count. **Do not press enter after entering the required node count**. When you are done with the selection, select **Submit** at the bottom of the page.
  
![Selecting Required Licenses]({{site.baseurl}}/images/screenshot 2021-08-19 21.40.44.png)


### Confirm License status 
  
After refreshing the page, the selected license and node count will be displayed on the WAE Smart Licensing UI. 
 

![Confirm license and count]({{site.baseurl}}/images/screenshot 2021-08-19 14.26.47.png)  

  
Running the **license_check** command on the WAE server will also show the licenses associated with this instance of WAE server.
  
```
[wae@wae7 ~]$ license_check

Product: WAE Design
==========================================================================================================

Feature                   Expiration Date           Licensed Nodes       Status
-----------------------------------------------------------------------------------------------------------
MD_LSPLoadshare           2021 Nov 17               50                   InCompliance
MD_Sim                    2021 Nov 17               50                   InCompliance
MD_Demands                2021 Nov 17               50                   InCompliance
MD_RSVP                   2021 Nov 17               50                   InCompliance
MD_BGP                    2021 Nov 17               50                   InCompliance
MD_Changeover             2021 Nov 17               50                   InCompliance
MD_Nodes                  2021 Nov 17               50                   InCompliance
MD_ArchiveUI              2021 Nov 17               50                   InCompliance
MD_Users                  2021 Nov 17               50                   InCompliance
MD_QoS                    2021 Nov 17               50                   InCompliance
MD_ParseConfigs           2021 Nov 17               50                   InCompliance
MD_DmdDeduct              2021 Nov 17               50                   InCompliance
MD_ExpOptTactical         2021 Nov 17               50                   InCompliance
MD_SimAnalysis            2021 Nov 17               50                   InCompliance
MD_SegmentRouting         2021 Nov 17               50                   InCompliance
MD_MetricOptTactical      2021 Nov 17               50                   InCompliance
MD_ParseIGPDB             2021 Nov 17               50                   InCompliance
MD_MetricOpt              2021 Nov 17               50                   InCompliance
MD_GUI                    2021 Nov 17               50                   InCompliance
MD_ExpOpt                 2021 Nov 17               50                   InCompliance

Product: WAE Collector
==========================================================================================================

Feature                   Expiration Date           Licensed Nodes       Status
-----------------------------------------------------------------------------------------------------------
MC_Users                  2021 Nov 17               50                   InCompliance
MC_BGP                    2021 Nov 17               50                   InCompliance
MC_Login                  2021 Nov 17               50                   InCompliance
MC_ParseIGPDB             2021 Nov 17               50                   InCompliance
MC_ParseConfigs           2021 Nov 17               50                   InCompliance
MC_QoS                    2021 Nov 17               50                   InCompliance
MC_LDP                    2021 Nov 17               50                   InCompliance
MC_SNMP                   2021 Nov 17               50                   InCompliance
MC_RSVP                   2021 Nov 17               50                   InCompliance
MC_Nodes                  2021 Nov 17               50                   InCompliance


Product: WAE Live
==========================================================================================================

Feature                   Expiration Date           Licensed Nodes       Status
-----------------------------------------------------------------------------------------------------------
ML_Users                  2021 Nov 17               50                   InCompliance


WAE Server: "198.18.134.30"
```
  
### Install WAE Design License
  
At this point, you should be able to install the WAE Design Smart License by connecting it to the WAE Server.
  
![WAE Design Smart License]({{site.baseurl}}/images/screenshot 2021-08-19 21.59.12.png)
  
After this, perform a license refresh and/or exit and restart the WAE Design Client. Doing a license check should show the correct licenses being installed.
  
![WAE Design Smart License Installed]({{site.baseurl}}/images/screenshot 2021-08-20 05.54.42.png)
  
Note: the WAE Server NETCONF port (2022 by default) must always be reachable by the WAE Design client for Smart Licensing to be used.  
  
  

  

