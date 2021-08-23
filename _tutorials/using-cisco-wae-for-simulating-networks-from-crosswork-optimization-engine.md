---
published: true
date: '2021-08-23 14:43 +0800'
title: >-
  Using Cisco WAE to run simulation on network models from Crosswork
  Optimization Engine
author: Fung Lim
excerpt: >-
  The Crosswork Optimization Engine provides a RESTCONF based API which allows
  for the real-time network model to be retrieved. The Optimization Engine and
  WAN Automation Engine uses the same schema which allows network models to be
  used interchangeably across the two platforms, allowing for a multitude of
  capacity management use cases to be addressed by these two products.
tags:
  - Crosswork
  - Crosswork Optimization Engine
  - WAN Automation Engine
  - Plan File
---
{% include toc %}

## Introduction

The Cisco Crosswork™ Optimization Engine is a key component of the Crosswork Automation Suite. It 
provides real-time network optimization capabilities that allow operators to effectively maximize network utility and increase service velocity using real-time traffic engineering and proactive optimization.

Crosswork Optimization Engine is an effective tool for creating network intent through the implementation and visualization of Segment Routing Traffic Engineering (SR-TE) policies and for addressing transient network congestion through Local Congestion Mitigation (LCM) and its use of Tactical Traffic Engineering (TTE) SR-TE policies.

The WAN Automation Engine (WAE) works hand-in-hand with Crosswork Optimization Engine to address  requirements of different aspects of capacity management, ranging from long-term network engineering to capacity planning and traffic engineering. This allows the network to operate at an optimum level at all times.

The WAN Automation Engine is also useful for simulation analysis, in determining potential network hotspots during failure scenarios. 

This tutorial demonstrates how the real-time network model in Crosswork Optimization Engine can be retrieved and used in WAN Automation Engine for capacity management and simulation analysis purposes.

## Crosswork Optimization Engine

The screenshot below shows a network managed by the Crosswork Optimization Engine. 

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/screenshot 2021-08-23 14.31.07.png)

The network comprise of devices (nodes, links) as well as Segment Routing Traffic Engineering (SR-TE) policies. Crosswork Optimization Engine provides the user with a real-time view of the topology and traffic on its web based user interface.

## Retrieving network model using Crosswork RESTCONF API

In order to retrieve the network model from Optimization Engine, we leverage the Crosswork RESTCONF APIs exposed by the Optimization Engine. The details of these RESTCONF APIs are documented on DevNet under [Optimization Engine Operations](https://developer.cisco.com/docs/crosswork/#!crosswork-optimization-engine-apis-2-0-release-apis-optimization-engine-operations).

![Plan file export using Crosswork RESTCONF API]({{site.baseurl}}/images/screenshot 2021-08-23 15.23.54.png)


The get-plan RESTCONF request allows the user to specify the schema version compatible with the WAE Design version in use (7.4 as above) and the plan file format (txt or pln). Regardless of the parameters specified, the plan file is returned as a base64 encoded field in JSON body. The user may use utilities such as base64 to decode the encoded plan file.

```
base64 --decode /tmp/encoded.txt > /tmp/decoded.txt
```

The following provides an example of a complete shell script to retrieve the optimization engine model using curl. The first two curl calls are used to retrieve the tgt_token and jwt_token which are required for authenticating the actual get-plan request.


```
#!/bin/sh

host='198.18.134.219'
tgt_token=`curl -k -X POST https://${host}:30603/crosswork/sso/v1/tickets \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Accept: text/plain' \
  -d 'username=admin&password=Password'`

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

The output of the curl command needs to be saved to file, parsed and decoded.


## WAN Automation Engine

Once the plan file has been decoded, WAE Design Client may be used to open the network model. Simulations can be performed on the plan file as though the file was retrieved from a WAE Automation Server.

![Opening Network Model from WAE Design Client ]({{site.baseurl}}/images/screenshot 2021-08-23 15.38.41.png)


For example, the following shows the worst case traffic view as a result of a simulation analysis being performed on the network model.

![WAE Design WC traffic view]({{site.baseurl}}/images/screenshot 2021-08-23 15.39.17.png)
