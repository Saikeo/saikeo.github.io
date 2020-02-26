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
# 4. NAT configuration on R1 and R3

NAT configuration on R1
```
interface ethernet 0/0
 ip nat outside
 exit

interface ethernet 0/1
 ip nat inside
 exit
ip nat inside source static network 192.168.1.0 192.168.10.0 /24
```

NAT configuration on R3
```
interface ethernet 0/0
 ip nat outside
 exit

interface ethernet 0/1
 ip nat inside
 exit
ip nat inside source static network 192.168.1.0 192.168.20.0 /24
```
# 4. IPSec configuration on R1 and R3
 IPSec configuration on R1
```
crypto isakmp policy 1
 encr aes
 hash sha
 authentication pre-share
 group 5
 lifetime 1800
 exit
crypto isakmp key Saikeo address 0.0.0.0        
crypto ipsec security-association lifetime seconds 1800
crypto ipsec transform-set saikeo-set esp-aes esp-sha-hmac 
 mode tunnel
 exit
access-list 100 permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
crypto map saikeo 10 ipsec-isakmp 
 set peer 192.168.23.3
 set transform-set saikeo-set 
 match address 100
 exit

interface ethernet 0/0
 crypto map saikeo
 exit
```

IPSec configuration on R2
```
crypto isakmp policy 1
 encr aes
 hash sha
 authentication pre-share
 group 5
 lifetime 1800
 exit
crypto isakmp key Saikeo address 0.0.0.0        
crypto ipsec security-association lifetime seconds 1800
crypto ipsec transform-set saikeo-set esp-aes esp-sha-hmac 
 mode tunnel
 exit
access-list 100 permit ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
crypto map saikeo 10 ipsec-isakmp 
 set peer 192.168.12.1
 set transform-set saikeo-set 
 match address 100
 exit

interface ethernet 0/0
 crypto map saikeo
 exit
```