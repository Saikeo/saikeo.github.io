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
# set system root-authentication plain-text-password
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
* Test to login via ssh
<p align = "center">
<img src = "https://i.imgur.com/Lcewn9E.png">
</p>
* Test to login via web broswer
<p align = "center">
<img src = "https://i.imgur.com/9ur61Nl.png">
</p>

## [](#header-2) 2. Authentication and authorization
* Add new user "saikeo" with the predefined login class "super-user".
```
# edit system
# set login user saikeo class super-user
# set login user saikeo authentication plain-text-password
# commit
```
* Add new custom login class as below:

<p align = "left">
<img src = "https://i.imgur.com/FmFEAqO.png">
</p>
```
# edit system login 
# set class saikeo-class permissions all allow-commands "configure private" deny-commands configure
# set class saikeo-class permissions clear allow-commands "(show system uptime)|(show system storage)|(show interfaces terse)"
# set class saikeo-class permissions view-configuration deny-commands "file delete"
# set user ronly class ronly-class authentication plain-text-password 
# set user saikeo1 class saikeo-class authentication plain-text-password
# set user restricted class restricted-class authentication plain-text-password
```
Verify: User saikeo1 won't be able to use "configure" command. Need to use "configure private" command instead.
<p align = "left">
<img src = "https://i.imgur.com/dIGneJo.png">
</p>
* Syslog configuration
