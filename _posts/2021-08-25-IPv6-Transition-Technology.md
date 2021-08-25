---
title: IPv6 Transition Technology
tags: [IPv6,Networking]
published: true
description: In this topic I am going to demonstrate IPv6 transition by using 6to4 technology on Huawei router NE40E V800R011C00SPC607B607. 
thumbnail: https://www-static.cdn-one.com/cmsimages/en_41-what-is-ipv6-illustration_%20ip-07-2x.png
---

<p align = "center">
<img src = "https://i.imgur.com/dWFlAxM.png">
</p>

In this LAB I am going to configure IPv6 transition technology by using 6to4 technology. In this LAB include 3 Huawei router and 2 Ubuntu desktop.

# 1. Configure interface on R1
```
system-view
sysname R1
interface Ethernet1/0/0
 undo shutdown
 ip address 200.1.1.1 255.255.255.0

interface Ethernet1/0/2
 undo shutdown
 ipv6 enable
 ipv6 address 2002:C801:101::FFFF/64
```
# 2. Configure interface on R2
```
system-view
sysname R2
interface Ethernet1/0/1
 undo shutdown
 ip address 200.2.2.2 255.255.255.0
 
interface Ethernet1/0/2
 undo shutdown
 ipv6 enable
 ipv6 address 2002:C802:202::FFFF/64
```
# 3. Configure interface on Internet
```
system-view
sysname Internet
#
interface Ethernet1/0/0
 undo shutdown
 ip address 200.1.1.2 255.255.255.0
 
interface Ethernet1/0/1
 undo shutdown
 ip address 200.2.2.1 255.255.255.0
```
 4. Configure static route on R1 and R2
To make R1 and R2 can reach to each other WAN, we have to configure static route on both device as below:

On R1
```
ip route-static 0.0.0.0 0.0.0.0 200.1.1.2
```
On R2
```
ip route-static 0.0.0.0 0.0.0.0 200.2.2.1
```
## [](#header-2) 5.Configure 6to4 tunnel on R1
Configure 6to4 tunnel 
```
interface Tunnel0/0/0
 ipv6 enable
 ipv6 address auto link-local
 tunnel-protocol ipv6-ipv4 6to4
 source 200.1.1.1
```
Configure IPv6 static route 
```
ipv6 route-static 2002:C802:202:: 64 Tunnel0/0/0
```
## [](#header-2) 5.Configure 6to4 tunnel on R2
Configure 6to4 tunnel
```
interface Tunnel0/0/0
 ipv6 enable
 ipv6 address auto link-local
 tunnel-protocol ipv6-ipv4 6to4
 source 200.2.2.2
```
Configure IPv6 static route
```
ipv6 route-static 2002:C801:101:: 64 Tunnel0/0/0
```