---
published: true
date: '2021-08-18 22:30 +0800'
title: Using dCloud to Access the WAN Automation Engine Demos
author: Josh Peters
excerpt: Using dCloud to Access the WAN Automation Engine Demos
tags:
  - cisco
  - WAE
  - Automation
  - Capacity Planning
---
{% include toc %}

## Overview

The WAN Automation Engine (WAE) is a software platform that provides multivendor and multilayer visibility and analysis for service provider and large enterprise networks. It plays a critical role in answering key questions of network resource availability by simulating an abstracted model of the network, and when appropriate can automate and simplify Traffic Engineering mechanisms such as RSVP-TE and Segment Routing.

Here on XR Docs, we are aiming to provide tutorials, blogs and other technical resources in order to help customers and partners understand how to effectively use WAE. Given that WAE is a product that requires a license, these tutorials will be using Cisco dCloud which provides a private demo environment.

This tutorial will show you how to launch a WAE demo on dCloud. 

**Note:** There are several WAE demos on dCloud, the tutorial will tell you which demo to launch and what plan file to use.
{: .notice--info}

## Prerequisites

1. Make sure you have an account on cisco.com. 
2. Using your cisco.com account try logging in to [dcloud.cisco.com](https://dcloud.cisco.com/).

![dCloud Login](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeDcloud-login.png)
{: .align-center}

## Switch to the Appropriate Datacenter

1. Select your user profile icon in the top right of the page.
2. Change the datacenter option to the datacenter outlined in the tutorial.

![dCloud Datacenter](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeDcloud-dc.png)
{: .align-center}

## Search for the Specified WAE Demo

1. Select the *Catalog* tab on the top
2. Search for the demo outlined in the tutorial.

![dCloud Search](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeDcloud-search.png)
{: .align-center}

## Schedule and Launch the WAE Demo

1. Follow the steps to schedule and start the WAE demo.

**Note:** It may take up to 20 minutes after the scheduled start time before the demo is ready to use.
{: .notice--info}

## Access the WAE Demo

1. Select the *My Dashboard* tab on the top.
2. Select My Sessions, then select View for the demo.

![dCloud View Session](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeDcloud-view.png)
{: .align-center}

There are two methods to access the demos. 

- Option 1 (suggested): Identify and select the workstation and open the link that says "Remote Desktop" to launch a VNC session to the workstation within your browser.

![dCloud View Workstation](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeDcloud-webvnc.png)
{: .align-center}

- Option 2: Use cisco anyconnect to connect to the demo. Then use your remote desktop client to connect to the workstation. To connect via anyconnect use the credentials specified under Session Details.

![dCloud Anyconnect](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeDcloud-ac.png)
{: .align-center}

## Launch WAE Design and Open a Network Model

1. On the workstation desktop, open the WAE Design application.
2. Select *File* -> *Open*
3. Browse to the samples directory and select **us_wan_l1.txt**
4. In the web browser of the workstation you may also navigtae to the tutorial on XR docs and download the plan file from the tutorial if specified.

![WAE Design Network Model](https://xrdocs.github.io/xrdocs-images/assets/tutorial-images/waeDcloud-openDesign.gif){: .align-center}
