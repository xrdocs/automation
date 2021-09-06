---
published: true
date: '2021-09-06 14:45 +0800'
title: Setting up Crosswork for Secure Zero Touch Provisioning (SZTP)
author: Fung Lim
tags:
  - Crosswork
  - Zero Touch Provisioning
  - Secure Zero Touch Provisioning
  - ZTP
  - SZTP
excerpt: >-
  This primer provides an introduction to setting up Crosswork for Secure Zero
  Touch Provisioning (SZTP)
---
{% include toc %}

## Introduction

The Crosswork ZTP application provides operators with the ability to provision networking devices remotely.

Using Crosswork ZTP, operators can ship factory-fresh devices to remote locations and connect them to the network without prior configuration. Crosswork ZTP can help upgrade the software image on these devices to a certified version and ensure that the desired Day0 configurations are applied.

Once configured, ZTP onboards the new device to the Cisco Crosswork device inventory. You can then use other Cisco Crosswork applications to monitor and manage the device.

Crosswork ZTP starting from release 2.0 supports Secure ZTP (SZTP) as per RFC 8572. Secure ZTP combines seamless automated devices bring up with security. Network devices can securely establish a connection with the ZTP server and authenticate the onboarding information that it receives. This process helps mitigate security risks that may be present during the provisioning of remote devices.

This tutorial provides a quick guide to get Secure ZTP up and running on Crosswork.

## Secure ZTP Lifecycle

The Secure ZTP device lifecycle begins in the Unprovisioned state. Once the ZTP process initiates, the device and the bootstrap server validate each other and the payload.

The two use the device SUDI, ownership vouchers, and device owner certificates for validation over a HTTP/TLS connection. If this is successful, the device entry transitions to the InProgress state. It connects to the network and starts downloading image and configuration files. The device remains in the InProgress state until it posts a Provisioning Error, ZTP Error, or Provisioned status to Cisco Crosswork.

Once the provisioning process completes successfully, the device will transition to the Provisioned state. Once provisioned, Cisco Crosswork onboards the device. If Cisco NSO is a Cisco Crosswork provider, Cisco NSO will onboard the device. Next, the device state changes to Onboarded. The device is now visible in inventory, and you can monitor and manage it like any other Cisco Crosswork network device.

If any of the validation steps fail, Secure ZTP posts a Provisioning Error. If the image or configuration code fails verification or installation, Secure ZTP posts a ZTP Error instead. Like Classic ZTP, Secure ZTP is successful when the device loads its image and/or configuration successfully, connects with Cisco Crosswork, and posts a Provisioned status. License consumption is the same as in Classic ZTP.

## Configuring Crosswork for Secure ZTP (SZTP)

### Install ZTP application

First, ensure that the ZTP application is installed and fully functional. You may verify the status of Crosswork applications by going to Administration > Crosswork Manager > Zero Touch Provisioning. The status indicator under the Zero Touch Provisioning should be in green and this indicates that the application is Healthy.

![Crosswork Manager Screenshot]({{site.baseurl}}/images/ztp_screen00.png)

## Upload ZTP certificates

The next step involves uploading the ZTP certificates onto Crosswork. 

![ztp_screen01.png]({{site.baseurl}}/images/ztp_screen01.png)





