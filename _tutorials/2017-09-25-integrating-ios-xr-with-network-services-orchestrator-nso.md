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
IOS-XR ZTP will download a script (exr-ztp-nso.sh) from the URL specified in option 67 (IPv4) or 59 (IPv6) of the DHCP offer (see [Working with ZTP](https://xrdocs.github.io/software-management/tutorials/2016-08-26-working-with-ztp/)) for more details

The exr-ztp-nso.sh script will perform the following tasks
-Download and load a PnP Agent container on the device using Docker
-Create a configuration for the agent
-Apply the required IOS-XR configuration
-Run the container

The PnP Agent will be in charge of notifying to NSO that the device has booted, doing a 4-way handshake. The PnP Server will first send a Day-0 configuration, the agent will apply it and the server will then register the device into the NSO CDB. After the third PnP Work Request, the PnP Server package triggers a sync-from and the reactive FASTMAP mechanism to deploy services.

### PnP Agent

PnP is a protocol that allows day-0 configuration of a device from a server that has a back-end configuration. The server identifies the device (or agent) with its serial number. _Please note that this implementation does not use PnP credentials and has not been tested using HTTPS._

The cisco-pnp-agent folder contains the sources of the container hosted at https://containers.cisco.com/repository/mtache/cisco-pnp-agent. The PnP Agent will look for a configuration file at /root/config/cisco-pnp-agent.conf inside the container. If you want to build or run this container locally, a build and run scripts are available.

PnP workflow that has been tested:

    PnP Hello 4-way handshake between server and agent.
    Agent sends PnP Work Request
    Server sends the block of configuration lines.
    Agent apply each line one by one and check for syntax error, incomplete command or commit error and sends a response to the server.

This PnP Agent has only been tested with the NSO PnP Server. It uses the cli-config-data-block format for the cli-config capability.
