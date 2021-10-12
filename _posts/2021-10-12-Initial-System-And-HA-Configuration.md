---
title: Initail System and High Availability Configuration
tags: [Juniper,Networking,Firewall,Secuirty]
published: true
description: In this topic I am going to demonstrate how to configure initail system and high availability on Juniper vSRX-NG on Junos 21.2R1.10. 
thumbnail: https://i.imgur.com/a1OH6eY.png
---

<p align = "center">
<img src = "https://i.imgur.com/a1OH6eY.png">
</p>

In this LAB I am going to configure initail system and high availability on Juniper. In this LAB include 2 Juniper vSRX-NG, two switch, 2 Ubuntu desktop and 1 Windows.

## [](#header-2) 1. Initial Configuration on Juniper vSRX-NG
```
> configure
# set system root-authentication encrypted-password yourpassword
# set system host-name vSRX1
# commit
```
