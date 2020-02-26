---
title: How to configure IPSec site to site with overlapping subnet
tags: [IPSec,Cisco,Network,VPN]
published: true
description: This is method how to configure IPSec site to site with overlapping subnet on Cisco IOS
thumbnail: https://i.imgur.com/bam8rgk.png
---

<p align = "center">
<img src = "https://i.imgur.com/bam8rgk.png">
</p>

This is method how to configure IPSec site to site with overlapping subnet on Cisco IOS and I will use the above topology to demonstrate this method.

# 1. Basic configuration on R1
```
interface Ethernet0/0
 ip address 192.168.12.1 255.255.255.0
 no shutdown
 exit
 
interface Ethernet0/1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit
 
ip route 0.0.0.0 0.0.0.0 192.168.12.2
```
# 2. Basic configuration on R2
```
interface Ethernet0/0
 ip address 192.168.12.2 255.255.255.0
 no shutdown
 exit
 
interface Ethernet0/1
 ip address 192.168.23.2 255.255.255.0
 no shutdown
 exit
```
# 3. Basic configuration on R3
```
interface Ethernet0/0
 ip address 192.168.23.3 255.255.255.0
 no shutdown
 exit
 
interface Ethernet0/1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit
 
ip route 0.0.0.0 0.0.0.0 192.168.23.2
```
