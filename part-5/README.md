# SRv6 based IPv4 L3VPN

In this section we are going to enable IPv4 L3VPN over SRv6, this will be done one the PE's

```
router bgp 65200
 address-family vpnv4 unicast
  vrf all
   segment-routing srv6
    locator myLoc1
    alloc mode per-vrf
   !
  !
 !
!
end
```

This will enable SRv6 Locator For All VRF Under VPNv4 AFI and you can start to see a SID attach to the local customer prefix

```
RP/0/RP0/CPU0:Y-PE1#show bgp vpnv4 unicast rd 4:2002 local-sids 
Sat Sep 21 04:38:59.064 UTC
BGP router identifier 2.0.201.4, local AS number 65200
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0
BGP main routing table version 30
BGP NSR Initial initsync version 3 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Local Sid                                   Alloc mode   Locator
Route Distinguisher: 4:2002 (default for vrf yellow)
Route Distinguisher Version: 30
*> 172.0.222.2/32     2001:db8:0:4:40::                           per-vrf      myLoc1
r>i172.0.222.3/32     NO SRv6 Sid                                 -            -
r>i172.222.6.0/24     NO SRv6 Sid                                 -            -

Processed 3 prefixes, 3 paths
```

Furthermore, you can see a new SID in the local table

```
RP/0/RP0/CPU0:Y-PE1#show segment-routing srv6 sid 
Sat Sep 21 04:41:16.063 UTC

*** Locator: 'myLoc1' *** 

SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  --------------------------------  ------------------  -----  --
2001:db8:0:4:1::            End (PSP/USD)     'default':1                       sidmgr              InUse  Y 
2001:db8:0:4:40::           End.DT4           'yellow'                          bgp-65200           InUse  Y
RP/0/RP0/CPU0:Y-PE1#show segment-routing srv6 sid 2001:db8:0:4:40:: detail 
Sat Sep 21 04:45:35.116 UTC

*** Locator: 'myLoc1' *** 

SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  --------------------------------  ------------------  -----  --
2001:db8:0:4:40::           End.DT4           'yellow'                          bgp-65200           InUse  Y 
  SID Function: 0x40
  SID context: { table-id=0xe0000001 ('yellow':IPv4/Unicast) }
  Locator: 'myLoc1'
  Allocation type: Dynamic
  Created: Sep 21 04:37:59.523 (00:07:35 ago)
```

The `End.DT4` represents Endpoint with decapsulation and IPv4 table lookup. We also going to enable SRv6 L3VPN on PE2 and shall see the remote customer route with a SID

```
RP/0/RP0/CPU0:Y-PE1#show bgp vpnv4 unicast rd 5:2003 received-sids 
Sat Sep 21 04:40:42.080 UTC
BGP router identifier 2.0.201.4, local AS number 65200
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0
BGP main routing table version 38
BGP NSR Initial initsync version 3 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop                            Received Sid
Route Distinguisher: 5:2003
Route Distinguisher Version: 34
*>i172.0.222.3/32     2620:fc7:2001::5
                                                          2001:db8:0:5:40::
*>i172.222.6.0/24     2620:fc7:2001::5
                                                          2001:db8:0:5:40::

Processed 2 prefixes, 2 paths
RP/0/RP0/CPU0:Y-PE1#
```

At this point we shall see the customer learning the romote routes

```
Y-CE2#show bgp ipv4 unicast summary 
BGP router identifier 172.0.222.2, local AS number 2000
BGP table version is 4, main routing table version 4
3 network entries using 432 bytes of memory
3 path entries using 252 bytes of memory
2/2 BGP path/bestpath attribute entries using 320 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1028 total bytes of memory
BGP activity 3/0 prefixes, 3/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.222.4.16    4        65200      50      54        4    0    0 00:44:56        2
Y-CE2#show bgp ipv4 unicast         
BGP table version is 4, local router ID is 172.0.222.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
              t secondary path, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>   172.0.222.2/32   0.0.0.0                  0         32768 i
 *>   172.0.222.3/32   172.222.4.16                           0 65200 ?
 *>   172.222.6.0/24   172.222.4.16                           0 65200 ?
```

However, reachability isn't there

```
Y-CE2#ping 172.0.222.3 source loopback 0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.0.222.3, timeout is 2 seconds:
Packet sent with a source address of 172.0.222.2 
.....
Success rate is 0 percent (0/5)
```

This is due to the exchange of SRv6 capability on the underlay. The SID is not routeable as we look for the cutomer route `172.0.222.2`

```
RP/0/RP0/CPU0:Y-PE1#show bgp vpnv4 unicast rd 5:2003 172.0.222.3
Sat Sep 21 04:54:06.312 UTC
BGP routing table entry for 172.0.222.3/32, Route Distinguisher: 5:2003
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker                  34           34
Last Modified: Sep 21 04:39:58.209 for 00:14:08
Paths: (1 available, best #1)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Not advertised to any peer
  Local
    2620:fc7:2001::5 (metric 20) from 2620:fc7:2001::6 (2.0.201.5)
      Received Label 0x400
      Origin incomplete, metric 2, localpref 100, valid, internal, best, group-best, import-candidate, not-in-vrf
      Received Path ID 0, Local Path ID 1, version 34
      Extended community: OSPF domain-id:0x5:0x000000002003 OSPF route-type:0:1:0x0 OSPF router-id:2.0.201.5 RT:65200:2003 
      Originator: 2.0.201.5, Cluster list: 2.0.201.6
      PSID-Type:L3, SubTLV Count:1
       SubTLV:
        T:1(Sid information), Sid:2001:db8:0:5::, Behavior:19, SS-TLV Count:1
         SubSubTLV:
          T:1(Sid structure):
```

Route table for SID `2001:db8:0:5::`

```
RP/0/RP0/CPU0:Y-PE1#show route ipv6 2001:db8:0:5::
Sat Sep 21 04:55:32.872 UTC

% Network not in table
```

SRv6 is a natively support over IPv6 and the SID itself use IPv6 addresses. This SID will then encapsulate in the packet soon will be the destination address in the packet header. Therefore it should be routeable and need to enable under IS-IS

```
router isis CORE
 address-family ipv6 unicast
  segment-routing srv6
   locator myLoc1
    level 2
   !
  !
 !
!
end
```
Now the locator are exchanged and advertise thru ISIS

```
RP/0/RP0/CPU0:Y-PE1#show isis database Y-PE2.00-00 verbose detail | i Locator
Sat Sep 21 04:59:08.685 UTC
  SRv6 Locator:   MT (Standard (IPv4 Unicast)) 2001:db8:0:5::/64 D:0 Metric: 1 Algorithm: 0
RP/0/RP0/CPU0:Y-PE1#show route ipv6 2001:db8:0:5::                
Sat Sep 21 04:58:19.235 UTC

Routing entry for 2001:db8:0:5::/64
  Known via "isis CORE", distance 115, metric 11, SRv6-locator, type level-2
  Installed Sep 21 04:55:56.769 for 00:02:22
  Routing Descriptor Blocks
    fe80::5200:ff:fe11:4, from 2620:fc7:2001::5, via GigabitEthernet0/0/0/1
      Route metric is 11
  No advertising protos.
```

Now you should able to ping the remote site

```
Y-CE2#ping 172.0.222.3 source loopback 0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.0.222.3, timeout is 2 seconds:
Packet sent with a source address of 172.0.222.2 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/8/9 ms
```

However, the traceroute shows it's running on IPv4

```
Y-CE2#traceroute 172.0.222.3 source loopback 0 numeric probe 1
Type escape sequence to abort.
Tracing the route to 172.0.222.3
VRF info: (vrf in name/id, vrf out name/id)
  1 172.222.4.16 10 msec
  2 2.201.3.17 16 msec
  3 172.222.6.28 [AS 65200] 9 msec
```

This still to be unknown to me and will shall dig deeper and share it once available.
