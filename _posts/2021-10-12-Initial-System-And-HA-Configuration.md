---
title: Initail System and High Availability Configuration on Juniper vSRX
tags: [Juniper,Networking,Firewall,Secuirty]
published: true
description: In this topic I am going to demonstrate how to configure initail system and high availability on Juniper vSRX-NG on Junos 21.2R1.10. 
thumbnail: https://i.imgur.com/a1OH6eY.png
---

<p align = "center">
<img src = "https://i.imgur.com/a1OH6eY.png">
</p>

In this LAB I am going to configure initail system and high availability on Juniper. In this LAB include 2 Juniper vSRX-NG, two switch, 2 Ubuntu desktop and 1 Windows.

## [](#header-2) 1. Initial Configuration on both Juniper vSRX-NG
* Configuring hostname
```
> configure
# set system root-authentication encrypted-password yourpassword
# set system host-name vSRX1
# commit
```
* Configuring an IP address for management and loopback interfaces
```
# set interfaces fxp0 unit 0 family inet address 192.168.1.1/24
# set interfaces lo0 unit 0 family inet address 1.1.1.1/32
# commit
```
* Enable the service
```
# edit system services
# set telnet 
# set ssh root-login allow 
# set web-management http
# set web-management https system-generated-certificate
# commit
```
* Limit the maximum connections per minutes. The following commands set the limit to maximum 5 connections per minute for SSH, Telnet and FTP.
```
# edit system services
# set telnet rate-limit 5
# set ssh rate-limit 5
# set ftp connection-limit 5
```
* Now test to login via ssh and web broswer
<p align = "center">
<img src = "https://i.imgur.com/Lcewn9E.png">
</p>

<p align = "center">
<img src = "https://i.imgur.com/9ur61Nl.png">
</p>