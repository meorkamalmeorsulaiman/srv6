# PE-CE Routing

In this section we will enable PE-CE routing for CE2 and CE3.

## PE1 - CE2

PE1

```bash
RP/0/RP0/CPU0:Y-PE1(config)#show
Sat Sep 21 03:56:44.434 UTC
Building configuration...
!! IOS XR Configuration 7.9.1
vrf yellow
 address-family ipv4 unicast
  import route-target
   65200:2002
  !
  export route-target
   65200:2002
  !
 !
 address-family ipv6 unicast
  import route-target
   65200:2002
  !
  export route-target
   65200:2002
  !
 !
!
interface GigabitEthernet0/0/0/2
 vrf yellow
 ipv4 address 172.222.4.16 255.255.255.0
 ipv6 address 2620:fc7:172:2224::16/64
 no shutdown
!
!
route-policy PASSALL
  pass
end-policy
!
router bgp 65200
 address-family vpnv4 unicast
 !
 address-family vpnv6 unicast
 !
 vrf yellow
  rd 4:2002
  address-family ipv4 unicast
  !
  neighbor 172.222.4.25
   remote-as 2000
   address-family ipv4 unicast
    route-policy PASSALL in
    route-policy PASSALL out
    as-override
   !
  !       
 !
!
end
```
We are yet to turn on v6 AFI and leave it at this moment, `as-override` used for later on this lab with same AS's for between the customer sites.

Below are the configurations for CE2

```bash
router bgp 2000
 bgp router-id 172.0.222.2
 bgp log-neighbor-changes
 neighbor 172.222.4.16 remote-as 65200
 !
 address-family ipv4
  network 172.0.222.2 mask 255.255.255.255
  neighbor 172.222.4.16 activate
 exit-address-family
```

We are not enable multihomed yet, as we trying to simplify this learning. Proceed to validate route learning for CE2

```bash
RP/0/RP0/CPU0:Y-PE1#show bgp vrf yellow         
Sat Sep 21 04:00:24.927 UTC
BGP VRF yellow, state: Active
BGP Route Distinguisher: 4:2002
VRF ID: 0x60000001
BGP router identifier 2.0.201.4, local AS number 65200
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000001   RD version: 5
BGP main routing table version 5
BGP NSR Initial initsync version 3 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 4:2002 (default for vrf yellow)
Route Distinguisher Version: 5
*> 172.0.222.2/32     172.222.4.25             0             0 2000 i
```

Thats all for CE2, let's proceed with CE3

## PE2 - CE3

PE2

```bash
RP/0/RP0/CPU0:Y-PE2(config)#show
Sat Sep 21 04:04:58.637 UTC
Building configuration...
!! IOS XR Configuration 7.9.1
vrf yellow
 address-family ipv4 unicast
  import route-target
   65200:2003
  !
  export route-target
   65200:2003
  !
 !
 address-family ipv6 unicast
  import route-target
   65200:2003
  !
  export route-target
   65200:2003
  !
 !
!
interface GigabitEthernet0/0/0/3
 description CE3
 vrf yellow
 ipv4 address 172.222.6.17 255.255.255.0
 ipv6 address 2620:fc7:172:2226::17/64
 no shutdown
!
router ospf 65200
 vrf yellow
  router-id 2.0.201.5
  domain-id type 0005 value 000000002003
  redistribute bgp 65200
  address-family ipv4 unicast
  area 0
   interface GigabitEthernet0/0/0/3
    network point-to-point
   !
  !
 !
!
router bgp 65200
 address-family vpnv4 unicast
 !
 address-family vpnv6 unicast
 !
 vrf yellow
  rd 5:2003
  address-family ipv4 unicast
   redistribute ospf 65200
  !
 !
!
end
```

CE3

```bash
router ospf 2000
 router-id 172.0.222.3
!
interface Loopback0
 ip ospf 2000 area 0
!
interface Ethernet0/0
 ip ospf network point-to-point
 ip ospf 2000 area 0
```

That's all and let's verify that the route learning correctly from OSPF to BGP

```bash
RP/0/RP0/CPU0:Y-PE2#show bgp vrf yellow 
Sat Sep 21 04:07:58.071 UTC
BGP VRF yellow, state: Active
BGP Route Distinguisher: 5:2003
VRF ID: 0x60000001
BGP router identifier 2.0.201.5, local AS number 65200
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000001   RD version: 10
BGP main routing table version 10
BGP NSR Initial initsync version 7 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 5:2003 (default for vrf yellow)
Route Distinguisher Version: 10
*> 172.0.222.3/32     172.222.6.28             2         32768 ?
*> 172.222.6.0/24     0.0.0.0                  0         32768 ?
```

That's all for PE-CE routing and let's move on with VPNv4 neighborship over the core.
