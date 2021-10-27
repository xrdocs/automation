---
published: true
date: '2021-10-27 10:17 +0800'
title: Using Cisco WAE for SRv6 and SRv6 TE visualization and simulation
author: Fung Lim
tags:
  - Crosswork
  - Crosswork Optimization Engine
  - WAE Design
  - WAN Automation Engine
  - SRv6
  - FlexAlgo
---
## Introduction

Cisco WAN Automation Engine release 7.5 introduces capabilities for modeling SRv6 and FlexAlgo. However, this initial phase relies on Crosswork Optimization Engine 3.0 for network collection and model building. In the tutorial [Using Cisco WAE to run simulation on network models](https://xrdocs.io/automation/tutorials/using-cisco-wae-to-run-sim-on-network-models-from-OE/) from Crosswork Optimization Engine, we provided an example on the steps required to retrieve the network model built using the Crosswork Optimization Engine.

Using the same steps, we should be able to retrieve a plan file (network model) comprising of SRv6 and FlexAlgo attributes from the network.

This tutorial provides an example of how such a model might render in the WAE Design client and how its representation compare with that on the Crosswork Optimization Engine.

## Crosswork Optimization Engine model

On the Crosswork Optimization Engine, a network with SRv6 and FlexAlgo is rendered on the user interface as follows. As can be seen below, Crosswork Optimization Engine allows the visualization of up to two user selected FlexAlgos on the topology map, although there might be more FlexAlgos deployed on the network.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img001.png)

If a node is selected with the SRv6 tab, details such as SRv6 Locators, relevant SIDs, and the Algorithms they participate in will be rendered on the user interface. 

Note: Maximum SID Depth (MSD) discovery information will only be reflected under **SR-MPLS > Prefixes** Tab.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img002.png)

Lastly, Crosswork Optimization Engine allows for the discovery and visualization of PCC-init SRv6 TE policies on the network, including rendering of details on candidate path. 

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img003.png)


## Using WAE Design to open Optimization Engine model

After we have successfully retrieve the Optimization Engine (OE) model from Crosswork Optimization Engine, we may use WAE Design Client to open the plan file. There are three additional tabs available - **SRv6 Node SIDs, SRv6 Interface SIDs, and Flex Algorithms.**

As can be seen below, the Node, SRv6 Locator, SR Algorithm and Protected attribute will be reflected on the SRv6 Node SID table.

Note: These additional tables may not be displayed by default, and the user needs to select these tables to be displayed explictly.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img004.png)

The SRv6 Interface SIDs table shows the same information but from the perspective of interfaces on the network. 

Note: The Interfaces table has also been augmented with additional fields including FlexAlgo affinities.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img005.png)

The Flex Algorithm Table provides a list of known Algorithms which have been collected via the SR-PCE, including additional attributes such as the Metric Type and affinity information (not shown).

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img006.png)

Under the LSP table, both SR-TE and SRv6 TE Policies will be displayed. One can select a SRv6 TE Policy to render the IGP path taken by that SRv6 TE Policy.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img007.png)



