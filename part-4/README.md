# VPNv4 Neighborship

In this section we are going to establish VPNv4 neighborship toward the RR for all the PE. These includes the ASBR's and PE's. The RR will be just reflecting the customer routes.


## Route-Reflector configs

The RR should establish VPNv4 neighborship to all the PE's

```bash
RP/0/RP0/CPU0:Y-RR#show configuration commit changes last 1
Sat Sep 21 04:16:18.784 UTC
Building configuration...
!! IOS XR Configuration 7.9.1
router bgp 65200
 bgp router-id 2.0.201.6
 ibgp policy out enforce-modifications
 address-family vpnv4 unicast
 !
 address-family vpnv6 unicast
 !
 neighbor 2620:fc7:2001::1
  remote-as 65200
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
 !
 neighbor 2620:fc7:2001::2
  remote-as 65200
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
 !
 neighbor 2620:fc7:2001::4
  remote-as 65200
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
 !
 neighbor 2620:fc7:2001::5
  remote-as 65200
  update-source Loopback0
  address-family vpnv4 unicast
   route-reflector-client
  !
 !
!
end
```

## PE's

In the previous section we haven't set the router-id and for the ASBR's you need to activate the address-family

PE's

```bash
router bgp 65200
 bgp router-id 2.0.201.5
 ibgp policy out enforce-modifications
 neighbor 2620:fc7:2001::6
  remote-as 65200
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
!
end
```

ASBR's

```bash
RP/0/RP0/CPU0:Y-ASBR2(config)#show
Sat Sep 21 04:20:11.899 UTC
Building configuration...
!! IOS XR Configuration 7.9.1
router bgp 65200
 bgp router-id 2.0.201.2
 ibgp policy out enforce-modifications
 address-family vpnv4 unicast
 !
 address-family vpnv6 unicast
 !
 neighbor 2620:fc7:2001::6
  remote-as 65200
  update-source Loopback0
  address-family vpnv4 unicast
  !
 !
!
end
```

## Validate

At this point, we should form the neighborship toward the RR and session should be establish

```bash
RP/0/RP0/CPU0:Y-RR#show bgp vpnv4 unicast summary 
Sat Sep 21 04:22:20.187 UTC
BGP router identifier 2.0.201.6, local AS number 65200
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0
BGP main routing table version 10
BGP NSR Initial initsync version 4 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker              10         10         10         10          10           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2620:fc7:2001::1
                  0 65200       5      14       10    0    0 00:02:23          0
2620:fc7:2001::2
                  0 65200       4      15       10    0    0 00:02:01          0
2620:fc7:2001::4
                  0 65200      10      15       10    0    0 00:00:43          1
2620:fc7:2001::5
                  0 65200      14      16       10    0    0 00:01:07          2

RP/0/RP0/CPU0:Y-RR#
```

We can see that the route being learn from the PE. But the egress PE wont able to import as the RT is unique for each side.

```bash
RP/0/RP0/CPU0:Y-PE1#show bgp vpnv4 unicast summary 
Sat Sep 21 04:22:45.759 UTC
BGP router identifier 2.0.201.4, local AS number 65200
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0
BGP main routing table version 5
BGP NSR Initial initsync version 3 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker               5          5          5          5           5           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2620:fc7:2001::6
                  0 65200       8       5        5    0    0 00:01:08          0
```

Let's import the RT for the remote side

PE1

```bash
RP/0/RP0/CPU0:Y-PE1(config)#show
Sat Sep 21 04:24:10.461 UTC
Building configuration...
!! IOS XR Configuration 7.9.1
vrf yellow
 address-family ipv4 unicast
  import route-target
   65200:2003
  !
 !
 address-family ipv6 unicast
  import route-target
   65200:2003
  !
 !
!
end
RP/0/RP0/CPU0:Y-PE1#show vrf yellow 
Sat Sep 21 04:24:29.711 UTC
VRF                  RD                  RT                         AFI   SAFI     
yellow               4:2002             
                                         import  65200:2002          IPV4  Unicast  
                                         import  65200:2003          IPV4  Unicast  
                                         export  65200:2002          IPV4  Unicast  
                                         import  65200:2002          IPV6  Unicast  
                                         import  65200:2003          IPV6  Unicast  
                                         export  65200:2002          IPV6  Unicast
```

PE2

```bash
RP/0/RP0/CPU0:Y-PE2(config)#show
Sat Sep 21 04:25:01.637 UTC
Building configuration...
!! IOS XR Configuration 7.9.1
vrf yellow
 address-family ipv4 unicast
  import route-target
   65200:2002
  !
 !
 address-family ipv6 unicast
  import route-target
   65200:2002
  !
 !
!
end

RP/0/RP0/CPU0:Y-PE2(config)#end
Uncommitted changes found, commit them before exiting(yes/no/cancel)? [cancel]:yes
RP/0/RP0/CPU0:Y-PE2#show vrf yellow 
Sat Sep 21 04:25:06.931 UTC
VRF                  RD                  RT                         AFI   SAFI     
yellow               5:2003             
                                         import  65200:2002          IPV4  Unicast  
                                         import  65200:2003          IPV4  Unicast  
                                         export  65200:2003          IPV4  Unicast  
                                         import  65200:2002          IPV6  Unicast  
                                         import  65200:2003          IPV6  Unicast  
                                         export  65200:2003          IPV6  Unicast
```

At this point the route should be there in the egress PE

```bash
RP/0/RP0/CPU0:Y-PE1#show bgp vpnv4 unicast rd 4:2002 172.0.222.3/32
Sat Sep 21 04:28:08.876 UTC
BGP routing table entry for 172.0.222.3/32, Route Distinguisher: 4:2002
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker                  12           12
Last Modified: Sep 21 04:26:27.209 for 00:01:41
Paths: (1 available, best #1, RIB-failure)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Not advertised to any peer
  Local
    2620:fc7:2001::5 (metric 20) from 2620:fc7:2001::6 (2.0.201.5)
      Received Label 24000 
      Origin incomplete, metric 2, localpref 100, valid, internal, best, group-best, import-candidate, imported
      Received Path ID 0, Local Path ID 1, version 12
      Extended community: OSPF domain-id:0x5:0x000000002003 OSPF route-type:0:1:0x0 OSPF router-id:2.0.201.5 RT:65200:2003 
      Originator: 2.0.201.5, Cluster list: 2.0.201.6
      Source AFI: VPNv4 Unicast, Source VRF: default, Source Route Distinguisher: 5:2003
```

However, this won't work because it' require VPN label `Received Label 24000`. We should move on to next part to enable SRv6 endpoint behaviour to support L3VPN service over SRv6.
