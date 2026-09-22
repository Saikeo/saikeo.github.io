---
title: Layer 3 Virtual Private Network (L3VPN) Lab
tags: [MPLS,L3VPN,CCIE]
published: true
description: Basic Lab for Layer 3 Virtual Private Network (L3VPN)
thumbnail: https://www.rtbrick.com/docs/techdocs/current/l3vpnug/_images/l3vpn_overview.png
---

Here is a complete, multi-stage blueprint to build a hands-on L3VPN Topology Lab. This setup is designed to be built in network emulators like GNS3, EVE-NG, or Cisco Modeling Labs (CML) using standard Cisco IOS/IOS-XE images (such as IOSv or CSR1000v/8000v).

<p align = "center">
<img src = "https://i.imgur.com/UuEjh7Z.png">
</p>

## [](#header-2) 1. IP Addressing Matrix:
```
WAN Links to CE:
PE-1 to CE1: 192.168.12.0/30 (PE-1 .1, CE-1 .2)
PE-2 to CE2: 192.168.45.0/30 (PE-2 .1, CE-2 .2)

Core Transit Links:
PE-1 to P-1: 10.1.12.0/24 (PE-1 .1, P-1 .2)
P-1 to P-2: 10.1.23.0/24 (P-1 .1, P-2 .2)
P-2 to PE-2: 10.1.34.0/24 (P-2 .1, PE-2 .2)

Provider Loopbacks (Loopback0):
PE-1: 1.1.1.1/32
P-1: 2.2.2.2/32 
P-2: 3.3.3.3/32 
PE-2: 4.4.4.4/32

Customer Edge Loopbacks (Loopback0):
CE-1: 192.168.1.1/32
CE-2: 192.168.2.2/32 
```
## [](#header-2) 2. Stage 1: The Provider Core Infrastructure (Underlay & MPLS)

Before running an L3VPN, the provider core must have flawless IP reachability between PE loopbacks, and MPLS must be activated so transport labels can be assigned via LDP.

**1. Configure the Underlay IGP (OSPF) on Core Devices**

   Activate OSPF across all provider devices to advertise Loopback0 interfaces and core physical links.

**PE-1 Configuration:**
```
router ospf 1
 router-id 1.1.1.1
 
interface GigabitEthernet2
 description To_P-1
 ip address 10.1.12.1 255.255.255.0
 ip ospf 1 area 0

interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ip ospf 1 area 0
```

**P-1 Configuration:**
```
router ospf 1
 router-id 2.2.2.2

interface GigabitEthernet1
 description To_P-2
 ip address 10.1.23.1 255.255.255.0
 ip ospf 1 area 0

interface GigabitEthernet2
 description To_PE-1
 ip address 10.1.12.2 255.255.255.0
 ip ospf 1 area 0

interface Loopback0
 ip address 2.2.2.2 255.255.255.255
 ip ospf 1 area 0
```

**P-2 Configuration:**
```
router ospf 1
 router-id 3.3.3.3
 
interface GigabitEthernet1
 description To_P-1
 ip address 10.1.23.2 255.255.255.0
 ip ospf 1 area 0

interface GigabitEthernet2
 description To_PE-2
 ip address 10.1.34.1 255.255.255.0
 ip ospf 1 area 0
 
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
 ip ospf 1 area 0
```

**PE-2 Configuration:**
```
router ospf 1
 router-id 4.4.4.4
 
interface GigabitEthernet2
 description To_P-2
 ip address 10.1.34.2 255.255.255.0
 ip ospf 1 area 0
 
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
 ip ospf 1 area 0
```

**2. Enable Label Distribution Protocol (MPLS LDP)**

   Turn on MPLS globally and under all core-facing physical interfaces.

**PE-1 Configuration:**
```
mpls ip
mpls ldp router-id Loopback0 force

interface GigabitEthernet2
 mpls ip
```
**P-1 Configuration:**
```
mpls ip
mpls ldp router-id Loopback0 force

interface GigabitEthernet1
 mpls ip

interface GigabitEthernet2
 mpls ip
```
**P-2 Configuration:**
```
mpls ip
mpls ldp router-id Loopback0 force

interface GigabitEthernet1
 mpls ip

interface GigabitEthernet2
 mpls ip
```
**PE-2 Configuration:**
```
mpls ip
mpls ldp router-id Loopback0 force

interface GigabitEthernet2
 mpls ip
```
(Ensure mpls ip is added under every physical core interface on all four provider routers).
## [](#header-2) 3. Stage 2: Creating the VRF Overlay
Now that the core can forward labels, build the isolated routing architecture on the edge nodes (PE-1 and PE-2).

**1. Define the Customer VRF and Bind Interfaces**

   Create the customer VRF table instance, stamp it with a distinct Route Distinguisher (RD), and determine import/export targets (RT).

**PE-1 Configuration:**
```
vrf definition Saikeo
 rd 65000:100
 address-family ipv4
  route-target export 65000:100
  route-target import 65000:100
 exit-address-family
!
interface GigabitEthernet2
 description TO_CE-1
 vrf forwarding Saikeo
 ip address 192.168.12.1 255.255.255.252
```
**PE-2 Configuration:**
```
vrf definition Saikeo
 rd 65000:100
 address-family ipv4
  route-target export 65000:100
  route-target import 65000:100
 exit-address-family
!
interface GigabitEthernet2
 description TO_CE-2
 vrf forwarding Saikeo
 ip address 192.168.45.1 255.255.255.252
```
**2. Configure MP-BGP Core Signaling (VPNv4)**

   Establish an internal BGP relationship between the PE nodes to securely trade customer VRF routes across the core backbone.

**PE-1 Configuration:**
```
router bgp 65000
 bgp log-neighbor-changes
 neighbor 4.4.4.4 remote-as 65000
 neighbor 4.4.4.4 update-source Loopback0
 !
 address-family vpnv4
  neighbor 4.4.4.4 activate
  neighbor 4.4.4.4 send-community both
 exit-address-family
```
**PE-2 Configuration:**
```
router bgp 65000
 bgp log-neighbor-changes
 neighbor 1.1.1.1 remote-as 65000
 neighbor 1.1.1.1 update-source Loopback0
 !
 address-family vpnv4
  neighbor 1.1.1.1 activate
  neighbor 1.1.1.1 send-community both
 exit-address-family
```

## [](#header-2) 4. Stage 3: PE-to-CE Routing Integration
The client must now exchange local subnets with the provider. For this lab blueprint, we implement standard eBGP as the PE-to-CE routing protocol.

**1. Setup the Customer Edge (CE) Routers**

   The CE routers are completely unaware of MPLS or VRFs; they run standard, native IP configurations.

**CE-1 Configuration:**
```
interface Loopback0
 ip address 192.168.1.1 255.255.255.255
!
interface GigabitEthernet1
 description TO_PE-1
 ip address 192.168.12.2 255.255.255.252
!
router bgp 64501
 neighbor 192.168.12.1 remote-as 65000
 neighbor 192.168.12.1 activate
 network 192.168.1.1 mask 255.255.255.255
```
**CE-2 Configuration:**
```
interface Loopback0
 ip address 192.168.2.2 255.255.255.255
!
interface GigabitEthernet1
 description TO_PE-2
 ip address 192.168.45.2 255.255.255.252
!
router bgp 64502
 neighbor 192.168.45.1 remote-as 65000
 neighbor 192.168.45.1 activate
 network 192.168.2.2 mask 255.255.255.255
```
**2. Configure PE Peers to Face Customer VRF Address Families**

   Configure the matching eBGP customer peering sessions within the appropriate VRF routing process on the provider edges.

**PE-1 Configuration:**
```
router bgp 65000
 address-family ipv4 vrf Saikeo
  neighbor 192.168.12.2 remote-as 64501
  neighbor 192.168.12.2 activate
 exit-address-family
```
**PE-2 Configuration:**
```
router bgp 65000
 address-family ipv4 vrf Saikeo
  neighbor 192.168.45.2 remote-as 64502
  neighbor 192.168.45.2 activate
 exit-address-family
```
## [](#header-2) 5. Lab Verification Checkpoints

Once the text configurations are applied, use these validation checks to ensure your data path is functional:

**1. Verify the Core Label Paths:**

   On PE-1, run show mpls ldp neighbors. Ensure peer sessions to core routers are fully up
<p align = "center">
<img src = "https://i.imgur.com/vtSypfv.png">
</p>

**2. Verify BGP VPNv4 Status:**

   On PE-1, run show bgp vpnv4 unicast all summary. You should see neighbor 4.4.4.4 with prefixes received (State/PfxRcd > 0).
<p align = "center">
<img src = "https://i.imgur.com/aIsCEH4.png">
</p>

**3. Inspect the VRF Routing Table:**

   On PE-1, run show ip route vrf CUST_A. You should dynamically learn the remote loopback route 192.168.2.2/32 via BGP.
<p align = "center">
<img src = "https://i.imgur.com/gPbbKAZ.png">
</p>

**4. End-to-End Validation:**

   Go to CE-1 and run a targeted validation test to confirm full end-to-end data transmission:

```
CE-1# ping 192.168.2.2 source 192.168.1.1
```
<p align = "center">
<img src = "https://i.imgur.com/aMAWHl7.png">
</p>

Congrats We can ping from CE-1 loopback0 to CE-2 loopback0.