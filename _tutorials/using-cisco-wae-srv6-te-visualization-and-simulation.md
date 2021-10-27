---
published: false
date: '2021-10-27 10:17 +0800'
title: Using Cisco WAE for SRv6 and SRv6 TE visualization and simulation
---
## Introduction

Cisco WAN Automation Engine release 7.5 introduces capabilities for modeling SRv6 and FlexAlgo. However, this initial phase relies on Crosswork Optimization Engine 3.0 for network collection and model building. In the tutorial [Using Cisco WAE to run simulation on network models](https://xrdocs.io/automation/tutorials/using-cisco-wae-to-run-sim-on-network-models-from-OE/) from Crosswork Optimization Engine, we provided an example on the steps required to retrieve the network model built using the Crosswork Optimization Engine.

Using the same steps, we should be able to retrieve a plan file (network model) comprising of SRv6 and FlexAlgo attributes from the network.

This tutorial provides an example of how such a model might render in the WAE Design client and how its representation compare with that on the Crosswork Optimization Engine.

## Crosswork Optimization Engine model

On the Crosswork Optimization Engine, a network with SRv6 and FlexAlgo is rendered on the user interface as follows. As can be seen below, Crosswork Optimization Engine allows the visualization of up to two user selected FlexAlgos on the topology map, although there might be more FlexAlgos deployed on the network.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img001.png)

If a node with SRv6 is selected,  details such as SRv6 Locators, relevant SIDs, and the Algorithms they participate in will be rendered on the user interface.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img002.png)

Lastly, Crosswork Optimization Engine allows for the discovery and visualization of PCC-init SRv6 TE policies on the network, including rendering of details on candidate path. 

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-flexalgo-img003.png)



