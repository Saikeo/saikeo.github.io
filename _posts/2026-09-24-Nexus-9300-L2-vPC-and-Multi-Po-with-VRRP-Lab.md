---
title: Nexus 9300 L2-vPC and Multi-Po with VRRP Lab
tags: [MPLS,L3VPN,CCIE]
published: true
description: Basic Lab for Layer 3 Virtual Private Network (L3VPN)
thumbnail: https://i.imgur.com/W7J3sT3.png
---

Here is a comprehensive, step-by-step hands-on lab guide based on your topology diagram. This workbook covers the complete initialization, feature activation, core vPC orchestration, and VRRP gateway redundancy configuration for your Cisco Nexus 9300 environment.

<p align = "center">
<img src = "https://i.imgur.com/G1TOVF7.jpeg">
</p>

## [](#header-2) 1. Feature Activation & Global Prerequisites

Before building any visual topologies, you must explicitly enable the structural software features inside Cisco NX-OS. Run these commands on both NXOS1 and NXOS2.

```
! Execute on both Core Switches (NXOS1 & NXOS2)
configure terminal
  feature vpc
  feature lacp
  feature interface-vlan
  feature vrrp
exit
```
## [](#header-2) 2. Management & vPC Peer-Keepalive Configuration

The vPC Peer-Keepalive link prevents split-brain scenarios by transmitting heartbeats outside the data plane. As shown in the topology, this runs through an isolated management switch (mgmt-SW) over the mgmt0 interfaces.

**NXOS1 Configuration:**
```
configure terminal
  interface mgmt0
    ip address 10.10.10.1/24
    no shutdown
exit
```

**NXOS2 Configuration:**
```
configure terminal
  interface mgmt0
    ip address 10.10.10.2/24
    no shutdown
exit
```

## [](#header-2) 3. Core vPC Domain & Peer-Link Configuration

The vPC Domain bounds the two peer switches together. The vPC Peer-Link (Po-9 over Eth1/1-2) acts as the synchronization plane for MAC tables and control-plane states.

**NXOS1 (Primary - Priority 150)**

```
configure terminal
  ! Define the vPC Domain
  vpc domain 100
    peer-keepalive destination 10.10.10.2 source 10.10.10.1
    role priority 150
    system-priority 4000
  exit

  ! Configure physical trunk links for Peer-Link
  interface Ethernet1/1-2
    description vPC_Peer_Link_Physical
    channel-group 9 mode active
    no shutdown
  exit

  ! Build the logical Peer-Link
  interface port-channel 9
    description vPC_Peer_Link_Logical
    switchport mode trunk
    vpc peer-link
    no shutdown
  exit
```

**NXOS2 (Secondary - Priority 120)**

```
configure terminal
  ! Define the vPC Domain
  vpc domain 100
    peer-keepalive destination 10.10.10.1 source 10.10.10.2
    role priority 120
    system-priority 4000
  exit

  ! Configure physical trunk links for Peer-Link
  interface Ethernet1/1-2
    description vPC_Peer_Link_Physical
    channel-group 9 mode active
    no shutdown
  exit

  ! Build the logical Peer-Link
  interface port-channel 9
    description vPC_Peer_Link_Logical
    switchport mode trunk
    vpc peer-link
    no shutdown
  exit

```

## [](#header-2) 4. Downstream Multi-Port-Channel Layout

To connect the access switches (Acc-Sw-1 through Acc-Sw-4), configure matching physical and logical vPC member bundles.

**NXOS1 Downstream Configuration:**

```
configure terminal
  ! --- Port-Channel 1 (Acc-Sw-1) ---
  interface Ethernet1/3
    channel-group 1 mode active
    no shutdown
  interface port-channel 1
    switchport mode trunk
    vpc 1
    no shutdown

  ! --- Port-Channel 2 (Acc-Sw-2) ---
  interface Ethernet1/4
    channel-group 2 mode active
    no shutdown
  interface port-channel 2
    switchport mode trunk
    vpc 2
    no shutdown

  ! --- Port-Channel 3 (Acc-Sw-3) ---
  interface Ethernet1/5
    channel-group 3 mode active
    no shutdown
  interface port-channel 3
    switchport mode trunk
    vpc 3
    no shutdown

  ! --- Port-Channel 4 (Acc-Sw-4) ---
  interface Ethernet1/6
    channel-group 4 mode active
    no shutdown
  interface port-channel 4
    switchport mode trunk
    vpc 4
    no shutdown
exit
```

**NXOS2 Downstream Configuration:**

```
configure terminal
  ! --- Port-Channel 1 (Acc-Sw-1) ---
  interface Ethernet1/3
    channel-group 1 mode active
    no shutdown
  interface port-channel 1
    switchport mode trunk
    vpc 1
    no shutdown

  ! --- Port-Channel 2 (Acc-Sw-2) ---
  interface Ethernet1/4
    channel-group 2 mode active
    no shutdown
  interface port-channel 2
    switchport mode trunk
    vpc 2
    no shutdown

  ! --- Port-Channel 3 (Acc-Sw-3) ---
  interface Ethernet1/5
    channel-group 3 mode active
    no shutdown
  interface port-channel 3
    switchport mode trunk
    vpc 3
    no shutdown

  ! --- Port-Channel 4 (Acc-Sw-4) ---
  interface Ethernet1/6
    channel-group 4 mode active
    no shutdown
  interface port-channel 4
    switchport mode trunk
    vpc 4
    no shutdown
exit
```

## [](#header-2) 5. VLAN, SVIs, & VRRP Gateway Deployment

Configure local Switched Virtual Interfaces (SVIs) and assign Virtual Router Redundancy Protocol (VRRP) VIPs matching the diagram specifications (.254).

**NXOS1 Core Layer SVI & VRRP:**

```
configure terminal
  vlan 10,20,30,40,50
  
  interface Vlan10
    ip address 10.10.1.1/24
    no shutdown
    vrrp 10
      address 10.10.1.254
      priority 150
      no shutdown

  interface Vlan20
    ip address 10.10.2.1/24
    no shutdown
    vrrp 20
      address 10.10.2.254
      priority 150
      no shutdown

  interface Vlan30
    ip address 10.10.3.1/24
    no shutdown
    vrrp 30
      address 10.10.3.254
      priority 150
      no shutdown

  interface Vlan40
    ip address 10.10.4.1/24
    no shutdown
    vrrp 40
      address 10.10.4.254
      priority 150
      no shutdown

  interface Vlan50
    ip address 10.10.5.1/24
    no shutdown
    vrrp 50
      address 10.10.5.254
      priority 150
      no shutdown
exit
```

**NXOS2 Core Layer SVI & VRRP:**

```
configure terminal
  vlan 10,20,30,40,50
  
  interface Vlan10
    ip address 10.10.1.2/24
    no shutdown
    vrrp 10
      address 10.10.1.254
      priority 120
      no shutdown

  interface Vlan20
    ip address 10.10.2.2/24
    no shutdown
    vrrp 20
      address 10.10.2.254
      priority 120
      no shutdown

  interface Vlan30
    ip address 10.10.3.2/24
    no shutdown
    vrrp 30
      address 10.10.3.254
      priority 120
      no shutdown

  interface Vlan40
    ip address 10.10.4.2/24
    no shutdown
    vrrp 40
      address 10.10.4.254
      priority 120
      no shutdown

  interface Vlan50
    ip address 10.10.5.2/24
    no shutdown
    vrrp 50
      address 10.10.5.254
      priority 120
      no shutdown
exit
```

## [](#header-2) 6. Downstream Access Layer Configuration Templates

These configurations apply to your four access switches (Acc-Sw-1 through Acc-Sw-4). These switches treat the two upstream Nexus parents as a single logical switch entity via standard LACP (IEEE 802.3ad).Note: For the access layer, templates are written for standard Cisco IOS/Cisco Catalyst switches commonly used in these topologies.

**Acc-Sw-1 Configuration:**
```
configure terminal
  ! Define data center access vlans
  vlan 10,20,30,40,50
  exit

  ! Configure physical member ports facing the Nexus core
  interface range ethernet 0/0-1
    description Uplinks_to_NXOS1_and_NXOS2
    switchport trunk encapsulation dot1q
    switchport mode trunk
    channel-group 1 mode active
    no shutdown
  exit

  ! Configure logical cross-switch channel group
  interface port-channel 1
    description Logical_Uplink_to_vPC_Domain
    switchport mode trunk
exit
```

**Acc-Sw-2 Configuration:**
```
configure terminal
  ! Define data center access vlans
  vlan 10,20,30,40,50
  exit

  ! Configure physical member ports facing the Nexus core
  interface range ethernet 0/0-1
    description Uplinks_to_NXOS1_and_NXOS2
    switchport trunk encapsulation dot1q
    switchport mode trunk
    channel-group 2 mode active
    no shutdown
  exit

  ! Configure logical cross-switch channel group
  interface port-channel 2
    description Logical_Uplink_to_vPC_Domain
    switchport mode trunk
exit
```

**Acc-Sw-3 Configuration:**
```
configure terminal
  ! Define data center access vlans
  vlan 10,20,30,40,50
  exit

  ! Configure physical member ports facing the Nexus core
  interface range ethernet 0/0-1
    description Uplinks_to_NXOS1_and_NXOS2
    switchport trunk encapsulation dot1q
    switchport mode trunk
    channel-group 3 mode active
    no shutdown
  exit

  ! Configure logical cross-switch channel group
  interface port-channel 3
    description Logical_Uplink_to_vPC_Domain
    switchport mode trunk
exit
```

**Acc-Sw-4 Configuration:**
```
configure terminal
  ! Define data center access vlans
  vlan 10,20,30,40,50
  exit

  ! Configure physical member ports facing the Nexus core
  interface range ethernet 0/0-1
    description Uplinks_to_NXOS1_and_NXOS2
    switchport trunk encapsulation dot1q
    switchport mode trunk
    channel-group 4 mode active
    no shutdown
  exit

  ! Configure logical cross-switch channel group
  interface port-channel 4
    description Logical_Uplink_to_vPC_Domain
    switchport mode trunk
exit
```

## [](#header-2) 7. Advanced Architecture Optimization (vPC Peer-Gateway)

In a standard deployment, if a downstream device routes packets using a router MAC address belonging to the secondary vPC switch (NXOS2), but sends the frame to the primary switch (NXOS1), NXOS1 will pass the traffic across the vPC peer-link to NXOS2 rather than routing it locally. This causes unnecessary utilization of the peer-link.Enabling vPC Peer-Gateway allows both NXOS1 and NXOS2 to act as the active gateway for each other's local MAC addresses. This ensures that whichever switch receives the routed frame will route it locally, bypassing the peer-link completely.

**NXOS1 Peer-Gateway Update:**
```
configure terminal
  vpc domain 100
    peer-gateway
exit
```

**NXOS2 Peer-Gateway Update:**
```
configure terminal
  vpc domain 100
    peer-gateway
exit
```

## [](#header-2) 8. Verification and Validation Playbook

To ensure everything is operational, execute the following confirmation matrix:

```
show vpc
show vpc brief
show vpc consistency-parameters global
show vrrp summary
```

<p align = "center">
<img src = "https://i.imgur.com/x9t30GD.png">
</p>

<p align = "center">
<img src = "https://i.imgur.com/6tQXlMK.png">
</p>

<p align = "center">
<img src = "https://i.imgur.com/zqJkU9A.png">
</p>

## [](#header-2) 9. Practical Failover Engineering Exercises

To master high-availability operations, replicate these two physical failure test scenarios inside your lab environment:

**Test Case A: Link Aggregation Resiliency**
1. Run a continuous ping trace from Vlan-10-PC (10.10.1.10) toward its virtual default gateway (10.10.1.254).
2. Administratively shut down interface Eth1/3 exclusively on NXOS1.
3. Observed Result: Data traffic instantly routes over Eth1/3 on NXOS2 across the active bundle. Zero ping drops should occur.

<p align = "center">
<img src = "https://i.imgur.com/amKt8NH.png">
</p>

**Test Case B: Gateway Failover Redundancy**
1. Perform a complete shutdown of NXOS1 or disable its structural internal SVI interfaces (shutdown on interface Vlan10).
2. Observed Result: NXOS2 immediately identifies the loss of master VRRP advertisements, transitioning state from Backup to Master. The continuous endpoint tracking traffic will gracefully recover within 1 to 2 seconds.


