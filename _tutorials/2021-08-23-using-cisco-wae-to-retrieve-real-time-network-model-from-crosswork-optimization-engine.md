---
published: false
date: '2021-08-23 14:43 +0800'
title: >-
  Using Cisco WAE to retrieve real-time network model from Crosswork
  Optimization Engine
---
{% include toc %}

## Introduction

The Cisco Crosswork™ Optimization Engine is a key component of the Crosswork Automation Suite. It 
provides real-time network optimization capabilities that allow operators to effectively maximize network utility and increase service velocity using real-time traffic engineering and proactive optimization.

Crosswork Optimization Engine is an effective tool for creating network intent through the implementation and visualization of Segment Routing Traffic Engineering (SR-TE) policies and for addressing transient network congestion through Local Congestion Mitigation (LCM). 

The WAN Automation Engine (WAE) works hand-in-hand with Crosswork Optimization Engine to address  requirements of different aspects of capacity management, ranging from long term network engineering to capacity planning and traffic engineering. The WAN Automation Engine is also useful for simulation analysis, in determining potential network hotspots during failure scenarios. 

This tutorial demonstrates how the real-time network model in Crosswork Optimization Engine can be retrieved and used in WAN Automation Engine for capacity management and simulation analysis purposes.

## Crosswork Optimization Engine

The screenshot below shows a network managed by the Crosswork Optimization Engine. 

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/screenshot 2021-08-23 14.31.07.png)

The network model in optimization comprise of devices (nodes, links) as well as Segment Routing Traffic Engineering (SR-TE) policies.

## Retrieving network model using Crosswork RESTCONF API

```
#!/bin/sh

host='198.18.134.219'
tgt_token=`curl -k -X POST https://${host}:30603/crosswork/sso/v1/tickets \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Accept: text/plain' \
  -d 'username=admin&password=C!sco12345'`

# Get JWT token:

jwt_token=`curl -k -X POST \
  https://${host}:30603/crosswork/sso/v1/tickets/$tgt_token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d service=https://cw.domain.com/app-dashboard`

# Use API:
curl -k -X POST \
  -H "Authorization: Bearer ${jwt_token}" -H "Content-Type:application/json"\
  https://${host}:30603/crosswork/nbi/optima/v1/restconf/operations/cisco-crosswork-optimization-engine-operations:get-plan \
  -d '{"input":{"version":"7.4","format":"pln"}}'

```

## WAN Automation Engine

![WAE Design Client UI]({{site.baseurl}}/images/screenshot 2021-08-23 14.42.45.png)




