# Enabling SRv6

We are going to enable the SRv6 on all the PE's, SRv6 use IPv6 address as the SID. In this lab we are going to use Full-length SID as the locator. Locator is a 64 bits long address that divided into 2:

- The SID block: Not more that 40 bits
- The Node Id: Not more that 24 bits

The SID block is a well known address space in a SRv6 domain, while Node ID is the designator node. Below are the example to enable SRv6 on PE-1:

```bash
segment-routing
 srv6
  locators
   locator myLoc1
    prefix 2001:db8:0:4::/64
   !
  !
 !
!
end
```

We can see that we are using /64 prefix as the locator. Below divide into details:

- SID Block: `2001:0db8:00::/40`

```bash
RP/0/RP0/CPU0:Y-PE1#show segment-routing srv6 manager | i Base
Sat Sep 21 03:36:31.684 UTC
    Base:
      SID Base Block: 2001:db8::/40
```

- Node Id: `XXXX:XXXX:XX00:0004::/64`

```bash
RP/0/RP0/CPU0:Y-PE1#show segment-routing srv6 manager | begin Summary
Sat Sep 21 03:37:29.658 UTC
Summary:
  Number of Locators: 1 (1 operational)
  Number of SIDs: 0 (0 stale)
  Max SID resources: 64000
  Number of free SID resources: 64000
  OOR:
    Thresholds (resources): Green 3200, Warning 1920
    Status: Resource Available
        History: (0 cleared, 0 warnings, 0 full)
    Block 2001:db8:0:4::/64:
        Number of SIDs free: 65470
        Max SIDs: 65470
        Thresholds: Green 3274, Warning 1965
        Status: Resource Available
            History: (0 cleared, 0 warnings, 0 full)
```

The node ID will be unique for each PE in the SRv6 domain as we can see on each PE with the /64 block 

```bash
RP/0/RP0/CPU0:Y-PE2#show segment-routing srv6 manager | begin Summary
Sat Sep 21 03:38:49.690 UTC
Summary:
  Number of Locators: 1 (1 operational)
  Number of SIDs: 0 (0 stale)
  Max SID resources: 64000
  Number of free SID resources: 64000
  OOR:
    Thresholds (resources): Green 3200, Warning 1920
    Status: Resource Available
        History: (0 cleared, 0 warnings, 0 full)
    Block 2001:db8:0:5::/64:
        Number of SIDs free: 65470
        Max SIDs: 65470
        Thresholds: Green 3274, Warning 1965
        Status: Resource Available
            History: (0 cleared, 0 warnings, 0 full)
```

To verify the locator configuration as below:

```bash
RP/0/RP0/CPU0:Y-PE1#show segment-routing srv6 locator myLoc1 detail 
Sat Sep 21 03:39:45.913 UTC
Name                  ID       Algo  Prefix                    Status   Flags   
--------------------  -------  ----  ------------------------  -------  --------
myLoc1                1        0     2001:db8:0:4::/64         Up               
  Interface: 
    Name: srv6-myLoc1
    IFH : 0x0000001c
    IPv6 address: 2001:db8:0:4::/64
  Number of SIDs: 0
  Created: Sep 21 03:29:46.339 (00:09:59 ago)
```

Verify the SRv6 local allocation as below:

```bash
RP/0/RP0/CPU0:Y-PE1#show segment-routing srv6 locator myLoc1 sid 
Sat Sep 21 03:42:09.652 UTC
SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  --------------------------------  ------------------  -----  --
2001:db8:0:4:1::            End (PSP/USD)     'default':1                       sidmgr              InUse  Y
```

That's all for this section, we move forward with enabling PE-CE routing in part 3.
