---
title: How to configure IPSec site to site with Pre-shared using IPv6
tags: [IPSec,Cisco,Network,VPN, IPv6]
published: true
description: This is method how to configure IPSec site to site with Pre-shared using IPv6 on Cisco IOS
thumbnail: https://i.imgur.com/uOq9kv2.png
---

<p align = "center">
<img src = "https://i.imgur.com/uOq9kv2.png">
</p>

This is method how to configure IPSec site to site with Pre-shared using IPv6 on Cisco IOS and I will use the above topology to demonstrate this method. In this post I will focus on technical and practical not theory.

# 1. Basic configuration on ISP
```
ipv6 unicast-routing

interface Ethernet0/0
 ipv6 address 12:1:1::1/48
 no shutdown
 exit

interface Ethernet0/1
 ipv6 address 23:1:1::1/48
 no shutdown
 exit
 ```
# 2. Basic configuration on Vientiane
```
ipv6 unicast-routing

interface Ethernet0/0
 ipv6 address 12:1:1::100/48
 no shutdown
 exit

interface Ethernet0/1
 ipv6 address 1:1:1::1/64
 no shutdown
 exit

ipv6 route ::/0 12:1:1::1
 ```
# 3. Basic configuration on Champasuk
```
ipv6 unicast-routing

interface Ethernet0/0
 ipv6 address 2:1:1::1/64
 no shutdown
 exit

interface Ethernet0/1
 ipv6 address 23:1:1::100/48
 no shutdown
 exit

ipv6 route ::/0 23:1:1::1
 ```

# + Verify connectivity between Vientiane and Champasuk
```
Vientiane#ping ipv6 23:1:1::100
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 23:1:1::100, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/20 ms
Vientiane#
```
# 4. OSPF configuration on Vientiane and Champasuk
```
* On Vientiane router

ipv6 router ospf 1
 router-id 1.1.1.1
 no shutdown

interface Ethernet0/1
 ipv6 ospf 1 area 0
```
```
* On Champasuk router

ipv6 router ospf 1
 router-id 2.2.2.2
 no shutdown

interface Ethernet0/0
 ipv6 ospf 1 area 0
```
# 5. IPSec configuration on Vientiane and Champasuk
```
* On Vientiane router

crypto isakmp policy 1
 encr aes
 hash sha
 authentication pre-share
 group 5
 lifetime 1800

crypto ipsec transform-set keo-set esp-aes esp-sha-hmac 
 mode tunnel
 exit

crypto keyring saikeo  
  pre-shared-key address ipv6 23:1:1::100/48 key saikeo

crypto isakmp profile saikeo
   keyring saikeo
   match identity address ipv6 23:1:1::100/48 

crypto ipsec profile saikeo
 set transform-set keo-set 
 set isakmp-profile saikeo
```
```
* On Champasuk router

crypto isakmp policy 1
 encr aes
 hash sha
 authentication pre-share
 group 5
 lifetime 1800

crypto ipsec transform-set keo-set esp-aes esp-sha-hmac 
 mode tunnel
 exit

crypto keyring saikeo  
  pre-shared-key address ipv6 12:1:1::100/48 key saikeo

crypto isakmp profile saikeo
   keyring saikeo
   match identity address ipv6 12:1:1::100/48 

crypto ipsec profile saikeo
 set transform-set keo-set 
 set isakmp-profile saikeo
```
# 5. Tunnel configuration on Vientiane and Champasuk
```
* On Vientiane router

interface Tunnel0
 ipv6 address 10:1:1::1/48
 ipv6 ospf 1 area 0
 tunnel source Ethernet0/0
 tunnel mode ipsec ipv6
 tunnel destination 23:1:1::100
 tunnel protection ipsec profile saikeo
```
```
* On Champasuk router

interface Tunnel0
 ipv6 address 10:1:1::2/48
 ipv6 ospf 1 area 0
 tunnel source Ethernet0/0
 tunnel mode ipsec ipv6
 tunnel destination 12:1:1::100
 tunnel protection ipsec profile saikeo
```
Now we have done configuration tunnel on both router and OSPF neighbor should be up if everything is configure correctly. 
# 6. Verify OSPF Neighbor on Vientiane and Champasuk
```
Vientiane#show ipv6 ospf neighbor 

            OSPFv3 Router with ID (1.1.1.1) (Process ID 1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
2.2.2.2           0   FULL/  -        00:00:32    22              Tunnel0

Vientiane#show ipv6 route ospf 
IPv6 Routing Table - default - 10 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       RL - RPL, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       la - LISP alt, lr - LISP site-registrations, ld - LISP dyn-eid
       lA - LISP away, a - Application
O   2:1:1::/64 [110/1010]
     via FE80::A8BB:CCFF:FE00:300, Tunnel0
O   10:1:1::2/128 [110/1000]
     via FE80::A8BB:CCFF:FE00:300, Tunnel0
Vientiane#
```
```
Champasuk#show ipv6 ospf neighbor 

            OSPFv3 Router with ID (2.2.2.2) (Process ID 1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
1.1.1.1           0   FULL/  -        00:00:35    22              Tunnel0

Champasuk#show ipv6 route ospf 
IPv6 Routing Table - default - 10 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       RL - RPL, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       la - LISP alt, lr - LISP site-registrations, ld - LISP dyn-eid
       lA - LISP away, a - Application
O   1:1:1::/64 [110/1010]
     via FE80::A8BB:CCFF:FE00:100, Tunnel0
O   10:1:1::1/128 [110/1000]
     via FE80::A8BB:CCFF:FE00:100, Tunnel0
Champasuk#
```
# 7. Verify IPSec on Vientiane and Champasuk
```
Vientiane#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status

IPv6 Crypto ISAKMP SA

 dst: 12:1:1::100
 src: 23:1:1::100
 state: QM_IDLE         conn-id:   1001 status: ACTIVE

Vientiane#show crypto ipsec sa 

interface: Tunnel0
    Crypto map tag: Tunnel0-head-0, local addr 12:1:1::100

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (::/0/0/0)
   remote ident (addr/mask/prot/port): (::/0/0/0)
   current_peer 23:1:1::100 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 147, #pkts encrypt: 147, #pkts digest: 147
    #pkts decaps: 145, #pkts decrypt: 145, #pkts verify: 145
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 12:1:1::100,
     remote crypto endpt.: 23:1:1::100
     plaintext mtu 1422, path mtu 1500, ipv6 mtu 1500, ipv6 mtu idb Ethernet0/0
     current outbound spi: 0x44F5BD15(1156955413)
     PFS (Y/N): N, DH group: none
```
```
Champasuk#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status

IPv6 Crypto ISAKMP SA

 dst: 12:1:1::100
 src: 23:1:1::100
 state: QM_IDLE         conn-id:   1001 status: ACTIVE

Champasuk#show crypto ipsec sa 

interface: Tunnel0
    Crypto map tag: Tunnel0-head-0, local addr 23:1:1::100

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (::/0/0/0)
   remote ident (addr/mask/prot/port): (::/0/0/0)
   current_peer 12:1:1::100 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 181, #pkts encrypt: 181, #pkts digest: 181
    #pkts decaps: 184, #pkts decrypt: 184, #pkts verify: 184
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 23:1:1::100,
     remote crypto endpt.: 12:1:1::100
     plaintext mtu 1422, path mtu 1500, ipv6 mtu 1500, ipv6 mtu idb Ethernet0/1
     current outbound spi: 0x1A11EA25(437381669)
     PFS (Y/N): N, DH group: none
```
# 8. Verify connectivity from Vientiane and Champasuk
```
Vientiane#ping ipv6 2:1:1::1 source ethernet 0/1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2:1:1::1, timeout is 2 seconds:
Packet sent with a source address of 1:1:1::1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/6 ms
Vientiane#
```
```
Champasuk#ping ipv6 1:1:1::1 source ethernet 0/1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1:1:1::1, timeout is 2 seconds:
Packet sent with a source address of 23:1:1::100
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/6 ms
Champasuk#
```

Everything is working fine now. We can connect to Champasuk from Vientiane. All packet travel between Vientiane and Champasuk have been encrypt.