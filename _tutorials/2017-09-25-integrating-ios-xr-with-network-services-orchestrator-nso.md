---
published: false
date: '2017-09-25 13:54 -0700'
title: Integrating IOS-XR with Network Services Orchestrator (NSO)
---
## Introduction

In this techtorial we will review a method to integrate IOS-XR with Cisco Network Services Orchestrator (NSO) using the ZTP feature of IOS XR and a docker container.
This method will allow the admistrator to automatically deploy a Day-0 configuration and register a device to NSO during the initial boot. NSO will then be able to initiate a connection to the device (through a Netconf NED or a CLI NED), execute a sync-from command and deploy a service thanks to reactive FASTMAP feature.
