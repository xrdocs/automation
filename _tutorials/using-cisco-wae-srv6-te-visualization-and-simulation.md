---
published: true
date: '2023-09-09 00:00 +0800'
title: Using Cisco WAE for SRv6 and SRv6 TE visualization and simulation
author: Fung Lim
excerpt: 'This tutorial introduces SRv6 and SRv6 TE capabilities on WAE 7.6'
tags:
  - Crosswork
  - Crosswork Optimization Engine
  - WAE Design
  - WAN Automation Engine
  - SRv6
  - FlexAlgo
---
## Introduction

Cisco WAN automation engine 7.6 supports SRv6 visualization using SR-PCE based network collection using topo-bgpls-xtc-nimo and lsp-pcep-xtc-nimo.

These enhancements provide capabilities for customers looking at the collection, visualization, and modeling of their SRv6-capable networks, including the ability to use SR LSP Optimizers. WAE Design has incorporated SRv6 modeling capabilities since 7.5 release and can be used to open network models comprising SRv6 attributes from the Crosswork Optimization Engine. 

This tutorial shows how the WAE Design client renders SRv6 network models and compares its representation with the Crosswork Optimization Engine.

## Crosswork Optimization Engine model

On the Crosswork Optimization Engine, a SRv6 based network is rendered on the Topology view as follows. 

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-img001.png)

If a node is selected with the SRv6 chevron expanded, details such as *IPv6 Router ID*, *SR FlexAlgo*, *Locator prefix* and *SIDs* will be rendered on the user interface.

Note: Maximum SID Depth (MSD) discovery information will only be reflected under **SR-MPLS > Prefixes** Tab.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-img002.png)

Lastly, Crosswork Optimization Engine allows for the discovery and visualization of PCC-init SRv6 policies on the network, including rendering of IGP path. 

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-img003.png)

## Using WAE Design on SRv6 models 

Two additional tables (*SRv6 Node SIDs*, *SRv6 Interface SIDs*) were introduced to support the modeling of SRv6. The *SRv6 Node SIDs* table provides visibility to the Node, SRv6 Locator, SR Algorithm, and Protected attribute, while the *SRv6 Interface SIDs* table provides Node, Interface, SID, and SR FlexAlgo attributes. 

Note: WAE Design does not display these tables by default. 

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-img004.png)

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-img005.png)

Note: The *interfaces* table has been augmented with additional fields including SR FlexAlgo.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-img006.png)

The *LSPs* table list SR and SRv6 policies on the network. One may select a specific SRv6 policy to render the IGP path and filter to Segment Lists, Segment List Hops, which will display the SRv6 SID Hops.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-img007.png)

SR LSP Optimizers are supported on SRv6 policies, in the same manner as SR policies. The following demonstrates the use of SR LSP Optimizers to minimize the TE Path metric of a selected SRv6 policy.

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-img008.png)

![Crosswork Optimization Engine UI]({{site.baseurl}}/images/using-wae-srv6-img009.png)

