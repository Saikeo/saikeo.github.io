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

This is method how to configure IPSec site to site with overlapping subnet on Cisco IOS and I will use the above topology to demonstrate this method. In this post I will focus on technical and practical not theory.

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

IPSec configuration on R3
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
# 5. Verify

Try to ping to 192.168.20.1 from PC1.
```
PC1> ping 192.168.20.1 -t
192.168.20.1 icmp_seq=1 timeout
84 bytes from 192.168.20.1 icmp_seq=2 ttl=254 time=1.778 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=254 time=1.377 ms
84 bytes from 192.168.20.1 icmp_seq=4 ttl=254 time=2.468 ms
84 bytes from 192.168.20.1 icmp_seq=5 ttl=254 time=1.349 ms
84 bytes from 192.168.20.1 icmp_seq=6 ttl=254 time=1.331 ms
84 bytes from 192.168.20.1 icmp_seq=7 ttl=254 time=1.605 ms

PC1>
```
We are able to ping and it working perfectly. Now we are to check our NAT.

```
R1#sh ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 192.168.10.2:25833 192.168.1.2:25833 192.168.20.1:25833 192.168.20.1:25833
icmp 192.168.10.2:26345 192.168.1.2:26345 192.168.20.1:26345 192.168.20.1:26345
icmp 192.168.10.2:26601 192.168.1.2:26601 192.168.20.1:26601 192.168.20.1:26601
icmp 192.168.10.2:26857 192.168.1.2:26857 192.168.20.1:26857 192.168.20.1:26857
icmp 192.168.10.2:27113 192.168.1.2:27113 192.168.20.1:27113 192.168.20.1:27113
icmp 192.168.10.2:27369 192.168.1.2:27369 192.168.20.1:27369 192.168.20.1:27369
icmp 192.168.10.2:27625 192.168.1.2:27625 192.168.20.1:27625 192.168.20.1:27625
--- 192.168.10.2       192.168.1.2        ---                ---
--- 192.168.10.0       192.168.1.0        ---                ---
R1#
```
NAT is working and able to translate. Next we are going to check IPSec.
```
R1#sh crypto ipsec sa

interface: Ethernet0/0
    Crypto map tag: saikeo, local addr 192.168.12.1

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (192.168.10.0/255.255.255.0/0/0)
   remote ident (addr/mask/prot/port): (192.168.20.0/255.255.255.0/0/0)
   current_peer 192.168.23.3 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 6, #pkts encrypt: 6, #pkts digest: 6
    #pkts decaps: 6, #pkts decrypt: 6, #pkts verify: 6
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 192.168.12.1, remote crypto endpt.: 192.168.23.3
     plaintext mtu 1438, path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/0
     current outbound spi: 0xB079E509(2960778505)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0xB08DEF80(2962091904)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 1, flow_id: SW:1, sibling_flags 80004040, crypto map: saikeo
        sa timing: remaining key lifetime (k/sec): (4202409/1572)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0xB079E509(2960778505)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 2, flow_id: SW:2, sibling_flags 80004040, crypto map: saikeo
        sa timing: remaining key lifetime (k/sec): (4202409/1572)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:
          
     outbound pcp sas:
R1#
```
IPSec is working and packet have been encrypt and decrypt normally.
