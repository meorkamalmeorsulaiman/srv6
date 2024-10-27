# SRv6

This repository is the learning notes and exploration of mine into segment routing v6. Below is the lab topology

![Lab Topology](https://github.com/meorkamalmeorsulaiman/srv6/blob/main/images/topology.jpg)

These are the loopbacks

| Hostname | Loopback v4 | Loopback v6          |
|----------|-------------|----------------------|
| P1       | 2.0.201.3   | 2620:fc7:2001::3     |
| PE1      | 2.0.201.4   | 2620:fc7:2001::4     |
| PE2      | 2.0.201.5   | 2620:fc7:2001::5     |
| RR       | 2.0.201.6   | 2620:fc7:2001::6     |
| ASBR1    | 2.0.201.1   | 2620:fc7:2001::1     |
| ASBR2    | 2.0.201.2   | 2620:fc7:2001::2     |
| CE2      | 172.0.222.2 | 2620:fc7:172:2001::2 |
| CE3      | 172.0.222.3 | 2620:fc7:172:2001::3 |

The idea is to build an L3-VPN service over SRv6. At the end of this lab we will be having L3 reachability between CE2 and CE3. We will devide this learning into several parts:

- Part 1: L3 Reachability over Loopback Addresses
- Part 2: Enabling SRv6
- Part 3: PE-CE Routing
- Part 4: VPNv4 Neighborship
- Part 5: SRv6 based IPv4 L3VPN
