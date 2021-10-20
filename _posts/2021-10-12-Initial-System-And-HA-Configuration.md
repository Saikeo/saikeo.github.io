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
* Add user with our new custom login class as below:

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
* Syslog Configuration
In this case my Syslog server is 192.168.2.10.
```
# edit system syslog 
# set user * any emergency
# set host 192.168.2.10 any emergency
# set source-address 1.1.1.1
# set file messages any critical
# set file interactive-commands interactive-commands info
# set file saikeo-policy-logs user info
# set file saikeo-policy-logs match SK_FLOW
# set file saikeo-policy-logs archive size 512k files 20
# set file authorization-logs authorization info
# set time-format year
```
* NTP Configuration
Configure time zoe.
```
# edit system
# set time-zone Asia/Vientiane
# edit ntp
# set server 192.168.2.10
# set source-address 1.1.1.1
# set authentication-key 1 type md5 value saikeo
# set server 192.168.2.10 key 1
# set trusted-key 1
```
* SNMP Configuration
```
# edit ntp
# set community saikeo clients 192.168.2.10/32
# set community saikeo authorization read-only
```
Configure SNMP to send authentification failures, hardware and environment, Link transitions and routing protocol to NMS 192.168.2.10
```
# set trap-group saikeo-group categories authentication chassis link routing
# set trap-group saikeo-group targets 192.168.2.10
```
Configure SNMP contact, description and location.
```
# set contact "Saikeo User"
# set description "Saikeo's Device"
# set location "Vientiane"
```
