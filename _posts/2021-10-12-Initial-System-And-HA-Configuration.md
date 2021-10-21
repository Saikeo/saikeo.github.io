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
# set class ronly-class permissions all allow-commands "configure private" deny-commands configure
# set class saikeo-class permissions clear allow-commands "(show system uptime)|(show system storage)|(show interfaces terse)"
# set class restricted-class permissions view-configuration deny-commands "file delete"

# set user ronly class ronly-class authentication plain-text-password 
# set user saikeo1 class saikeo-class authentication plain-text-password
# set user restricted class restricted-class authentication plain-text-password
```
Verify: User saikeo1 won't be able to use "configure" command. Need to use "configure private" command instead.
<p align = "left">
<img src = "https://i.imgur.com/dIGneJo.png">
</p>
## 3. Syslog Configuration
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
## 4. NTP Configuration
Configure time zone, NTP server and authentication.
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
## 5. SNMP Configuration
```
# edit ntp
# set community saikeo clients 192.168.2.10/32
# set community saikeo authorization read-only
```
* Configure SNMP to send authentification failures, hardware and environment, Link transitions and routing protocol to NMS 192.168.2.10
```
# set trap-group saikeo-group categories authentication chassis link routing
# set trap-group saikeo-group targets 192.168.2.10
```
* Configure SNMP contact, description and location.
```
# set contact "Saikeo User"
# set description "Saikeo's Device"
# set location "Vientiane"
```
## 6. Creating Clusters
Configure cluster node ID for both node. O for node vSRX1 and 1 for node vSRX2.
```
root@vSRX1> set chassis cluster cluster-id 1 node 0 reboot
root@vSRX2> set chassis cluster cluster-id 1 node 1 reboot
```
Verify:

```javascript
{primary:node0}
root@vSRX1> show chassis cluster status 
Cluster ID: 1
Node   Priority Status               Preempt Manual   Monitor-failures

Redundancy group: 0 , Failover count: 1
node0  1        primary              no      no       None           
node1  1        secondary            no      no       None           
```
Configure GE-0/0/1 for Control link and GE-0/0/2 for Fabric link.
```
# set interfaces fab0 fabric-options member-interfaces ge-0/0/2
# set interfaces fab1 fabric-options member-interfaces ge-7/0/2
# set chassis cluster control-link-recovery
```
Define node0 and node1 specific parameters on this cluster.
```
# edit groups node0 
# set system host-name vSRX1
# set interfaces fxp0 unit 0 family inet address 192.168.1.1/24

# edit groups node1 
# set system host-name vSRX2
# set interfaces fxp0 unit 0 family inet address 192.168.1.2/24

# set apply-groups "${node}"
```
Configure the loopback address in standalone mode.
```
# set interfaces lo0 unit 0 family inet address 1.1.1.1/32
```
## 7. Configuring redundancy groups and ethernet interfaces
Set the node 0 with higher priority to make node 0 be primary for this cluster. 
```
# edit chassis cluster
# set redundancy-group 0 node 0 priority 200
# set redundancy-group 0 node 1 priority 100
```
Configure reth interface, number and add child interface into it. I am going to configure 2 reth interface with each 2 child interface.
```
# edit chassis cluster
# set reth-count 2
# top edit interfaces
# set ge-0/0/0 gigether-options redundant-parent reth0
# set ge-7/0/0 gigether-options redundant-parent reth0
# set ge-0/0/3 gigether-options redundant-parent reth1
# set ge-7/0/1 gigether-options redundant-parent reth1
```
I have create 3 VLAN on Dist-SW as below:
```
Dist-SW#sh vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1, Et1/3
10   Users                            active    Et0/2
20   DMZ                              active    Et0/3
30   Admin                            active    
```
Configure redundancy group.
```
# edit chassis cluster 
# set redundancy-group 1 node 0 priority 200
# set redundancy-group 1 node 1 priority 100
# set redundancy-group 1 preempt
# set redundancy-group 1 interface-monitor ge-0/0/0 weight 255
# set redundancy-group 1 interface-monitor ge-7/0/0 weight 255

# set redundancy-group 2 node 0 priority 100
# set redundancy-group 2 node 1 priority 200
# set redundancy-group 2 preempt
# set redundancy-group 2 interface-monitor ge-0/0/3 weight 255
# set redundancy-group 2 interface-monitor ge-7/0/1 weight 255
```
So reth0 is an IP interface without VLAN tagging and the reth1 is VLAN tagged interfaces.
```
# edit interfaces
# set reth0 redundant-ether-options redundancy-group 1
# set reth1 redundant-ether-options redundancy-group 2
# set reth0 unit 0 family inet address 192.168.255.2/24
# set reth1 vlan-tagging
# set reth1 unit 10 vlan-id 10
# set reth1 unit 10 family inet address 172.16.1.1/24
# set reth1 unit 20 vlan-id 20
# set reth1 unit 20 family inet address 172.16.2.1/24
# set reth1 unit 30 vlan-id 30
# set reth1 unit 30 family inet address 172.16.3.1/24
```