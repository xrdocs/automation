---
published: false
date: '2017-09-25 13:54 -0700'
title: Integrating IOS-XR with Network Services Orchestrator (NSO)
author: pwariche
excerpt: integrating IOS-Xr with NSO using ZTP and docker container
tags:
  - iosxr
  - cisco
  - docker
  - NSO
  - PnP
---
## Introduction

In this techtorial we will review a method to integrate IOS-XR with Cisco Network Services Orchestrator (NSO) using the ZTP feature of IOS XR and a docker container.
This method will allow the admistrator to automatically deploy a Day-0 configuration and register a device to NSO during the initial boot. NSO will then be able to initiate a connection to the device (through a Netconf NED or a CLI NED), execute a sync-from command and deploy a service thanks to reactive FASTMAP feature.

## Operation Flow
IOS-XR ZTP will download a script from the URL specified in option 67 (IPv4) or 59 (IPv6) of the DHCP offer (see [Working with ZTP](https://xrdocs.github.io/software-management/tutorials/2016-08-26-working-with-ztp/)

The built-in ZTP feature will run the DHCP client on the IOS XR at boot, retrieve the script URL through option 67 (IPv4) or 59 (IPv6), download it and execute it. This script (pre-agg.sh in our demo) will do the following tasks:

    Download and load a PnP Agent container on the device using Docker
    Creating a configuration for the agent, apply XR configuration commands required
    Run the container

The PnP Agent will be in charge of notifying to NSO that the device has booted, doing a 4-way handshake. The PnP Server will first send a Day-0 configuration, the agent will apply it and the server will then register the device into NSO CDB. After the third PnP Work Request, the PnP Server package triggers a sync-from and the reactive FASTMAP mechanism to deploy services.
