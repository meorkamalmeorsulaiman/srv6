# L3 Reachability over Loopback Addresses

The idea for this section is to build loopback reachability between all the routers within the topology. These loopbacks will then use to establish any of the overlay protocol that we will be using

### IS-IS L2

To simplify things, we will be using IS-IS L2 for the entire domain. At the time of writing only IS-IS is supported for SRv6

```bash
RP/0/RP0/CPU0:Y-P1#show configuration commit changes last 1
Fri Sep 20 13:29:34.944 UTC
Building configuration...
!! IOS XR Configuration 7.9.1
group CCIE-ISIS
 router isis '.*'
  is-type level-2-only
  address-family ipv4 unicast
   metric-style wide
   mpls ldp auto-config
  !
  address-family ipv6 unicast
   metric-style wide
   single-topology
  !
  interface 'Loopback0'
   address-family ipv4 unicast
   !
   address-family ipv6 unicast
   !
  !
  interface 'Giga.*'
   point-to-point
   address-family ipv4 unicast
   !
   address-family ipv6 unicast
   !      
  !
 !
end-group
router isis CORE
 apply-group CCIE-ISIS
 net 49.0002.0000.0201.0003.00
 address-family ipv4 unicast
 !
 address-family ipv6 unicast
 !
 interface Loopback0
 !
 interface GigabitEthernet0/0/0/0
 !
 interface GigabitEthernet0/0/0/1
 !
 interface GigabitEthernet0/0/0/2
 !
 interface GigabitEthernet0/0/0/3
 !
 interface GigabitEthernet0/0/0/4
 !
!         
end
```

We make sure that we have the correct neighborship and routes are learnt correctly

```bash
RP/0/RP0/CPU0:Y-P1#show isis neighbors 
Fri Sep 20 13:30:20.101 UTC

IS-IS CORE neighbors:
System Id      Interface        SNPA           State Holdtime Type IETF-NSF
Y-RR           Gi0/0/0/4        *PtoP*         Up    29       L2   Capable 
Y-PE1          Gi0/0/0/2        *PtoP*         Up    25       L2   Capable 
Y-PE2          Gi0/0/0/3        *PtoP*         Up    25       L2   Capable 
Y-ASBR1        Gi0/0/0/0        *PtoP*         Up    28       L2   Capable 

Total neighbor count: 4
```
The /128 routes

```bash
RP/0/RP0/CPU0:Y-P1#show route ipv6 isis | i /128 
Fri Sep 20 13:30:53.106 UTC
i L2 2620:fc7:2001::1/128 
i L2 2620:fc7:2001::2/128 
i L2 2620:fc7:2001::4/128 
i L2 2620:fc7:2001::5/128 
i L2 2620:fc7:2001::6/128
RP/0/RP0/CPU0:Y-P1#ping 2620:fc7:2001::1 source loopback 0
Fri Sep 20 13:31:34.081 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2620:fc7:2001::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/15/50 ms
RP/0/RP0/CPU0:Y-P1#
```

We are done at for this part
