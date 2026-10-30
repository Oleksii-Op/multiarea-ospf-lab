### Phase 3: Routes

1. Every prefix in section 3, except `192.168.122.0/24`, is installed on every router that does not own it.
2. R1 is the only source of the default route.
3. Fill the cost table from the formula in section 1 before you verify. Assume GigabitEthernet on every lab link.

| Link | Predicted cost | Verified cost |
|---|----------------|---|
| R1 Gi0/0 (and the other two ports on 10.10.1.0/29) |                | |
| R2 Gi0/1 – R3 Gi3/0 | 10             | |
| R2 Gi0/3 – R4 Gi0/1 | 10             | |
| R3 Gi0/0 – R9 Gi0/0 | 10             | |
| R4 Gi0/0 (server LAN) | 0              | |

| From | To | Predicted path (router hops) | Verified path |
|---|---|------------------------------|---|
| web-server-2 | db-server | 3                            | |
| web-server-2 | web-server-1 | 4                            | |
| R5 loopback | web-server-1 | 3                            | |
| db-server | a public address | 3                            | |

**Theory questions.**

- R2 and R3 are joined twice inside Area 0. After you have computed the costs, which path do you expect from R4 toward web-server-1, and what changes when you shut exactly one of those two links? Put the link back when you have the answer.
    There 2 routes from R2 to R9 in the routing table, so ECMP will take place. After disabling one of the interfaces, there will be only 1 entry in the routing table.
- Should R1 keep advertising the default after Gi2/0 loses its DHCP address? Describe the failure mode you want, and the failure mode you want to avoid.
- `5.5.5.5/32` has no LAN behind it. Which area owns that prefix, and which OSPF route type do you expect to see for it on R9?
        Area 1 owns that address, IA - OSPF inter area route.
**Prove it.** The routing table on R4, R5 and R9, showing remote server LANs, remote loopbacks, and a default. A traceroute from web-server-2 to web-server-1. The two worksheets.

```shell
root@web-server-1:~# traceroute 192.168.2.6
traceroute to 192.168.2.6 (192.168.2.6), 30 hops max, 60 byte packets
 1  _gateway (10.200.1.1)  18.410 ms  18.328 ms  18.306 ms
 2  10.200.0.1 (10.200.0.1)  48.456 ms  48.435 ms  48.413 ms
 3  10.10.1.3 (10.10.1.3)  78.542 ms  78.520 ms  78.513 ms
 4  192.168.1.6 (192.168.1.6)  78.478 ms  78.422 ms  78.412 ms
 5  192.168.2.6 (192.168.2.6)  78.377 ms  78.356 ms  78.335 ms
```

It takes path from web-server1 -> R9 -> R3 G0/2 -> G0/2 R2 -> R4 -> web-server-2

R4
```shell
Gateway of last resort is 192.168.1.5 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 192.168.1.5, 01:45:08, GigabitEthernet0/1
      1.0.0.0/32 is subnetted, 1 subnets
O IA     1.1.1.1 [110/21] via 192.168.1.5, 01:44:48, GigabitEthernet0/1
      2.0.0.0/32 is subnetted, 1 subnets
O IA     2.2.2.2 [110/11] via 192.168.1.5, 01:45:23, GigabitEthernet0/1
      3.0.0.0/32 is subnetted, 1 subnets
O IA     3.3.3.3 [110/21] via 192.168.1.5, 01:45:23, GigabitEthernet0/1
      4.0.0.0/32 is subnetted, 1 subnets
C        4.4.4.4 is directly connected, Loopback0
      5.0.0.0/32 is subnetted, 1 subnets
O        5.5.5.5 [110/21] via 192.168.1.5, 01:26:32, GigabitEthernet0/1
      9.0.0.0/32 is subnetted, 1 subnets
O IA     9.9.9.9 [110/31] via 192.168.1.5, 01:45:23, GigabitEthernet0/1
      10.0.0.0/8 is variably subnetted, 5 subnets, 3 masks
O IA     10.0.1.0/24 [110/20] via 192.168.1.5, 01:45:23, GigabitEthernet0/1
O IA     10.10.1.0/29 [110/20] via 192.168.1.5, 01:45:23, GigabitEthernet0/1
O IA     10.20.1.0/30 [110/20] via 192.168.1.5, 01:45:23, GigabitEthernet0/1
O IA     10.200.0.0/30 [110/30] via 192.168.1.5, 01:45:23, GigabitEthernet0/1
O IA     10.200.1.0/24 [110/40] via 192.168.1.5, 01:45:23, GigabitEthernet0/1
      192.168.1.0/24 is variably subnetted, 3 subnets, 2 masks
O        192.168.1.0/30 [110/20] via 192.168.1.5, 01:58:50, GigabitEthernet0/1
C        192.168.1.4/30 is directly connected, GigabitEthernet0/1
L        192.168.1.6/32 is directly connected, GigabitEthernet0/1
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/24 is directly connected, GigabitEthernet0/0
L        192.168.2.1/32 is directly connected, GigabitEthernet0/0
```

R5
```shell
Gateway of last resort is 192.168.1.1 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 192.168.1.1, 01:45:36, GigabitEthernet0/0
      1.0.0.0/32 is subnetted, 1 subnets
O IA     1.1.1.1 [110/21] via 192.168.1.1, 01:45:16, GigabitEthernet0/0
      2.0.0.0/32 is subnetted, 1 subnets
O IA     2.2.2.2 [110/11] via 192.168.1.1, 01:45:51, GigabitEthernet0/0
      3.0.0.0/32 is subnetted, 1 subnets
O IA     3.3.3.3 [110/21] via 192.168.1.1, 01:45:51, GigabitEthernet0/0
      4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/21] via 192.168.1.1, 01:55:09, GigabitEthernet0/0
      5.0.0.0/32 is subnetted, 1 subnets
C        5.5.5.5 is directly connected, Loopback0
      9.0.0.0/32 is subnetted, 1 subnets
O IA     9.9.9.9 [110/31] via 192.168.1.1, 01:45:51, GigabitEthernet0/0
      10.0.0.0/8 is variably subnetted, 5 subnets, 3 masks
O IA     10.0.1.0/24 [110/20] via 192.168.1.1, 01:45:51, GigabitEthernet0/0
O IA     10.10.1.0/29 [110/20] via 192.168.1.1, 01:45:51, GigabitEthernet0/0
O IA     10.20.1.0/30 [110/20] via 192.168.1.1, 01:45:51, GigabitEthernet0/0
O IA     10.200.0.0/30 [110/30] via 192.168.1.1, 01:45:51, GigabitEthernet0/0
O IA     10.200.1.0/24 [110/40] via 192.168.1.1, 01:45:51, GigabitEthernet0/0
      192.168.1.0/24 is variably subnetted, 3 subnets, 2 masks
C        192.168.1.0/30 is directly connected, GigabitEthernet0/0
L        192.168.1.2/32 is directly connected, GigabitEthernet0/0
O        192.168.1.4/30 [110/20] via 192.168.1.1, 01:55:09, GigabitEthernet0/0
O     192.168.2.0/24 [110/30] via 192.168.1.1, 01:55:09, GigabitEthernet0/0
```

R9
```shell
Gateway of last resort is 10.200.0.1 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.200.0.1, 01:46:07, GigabitEthernet0/0
      1.0.0.0/32 is subnetted, 1 subnets
O IA     1.1.1.1 [110/21] via 10.200.0.1, 01:46:17, GigabitEthernet0/0
      2.0.0.0/32 is subnetted, 1 subnets
O IA     2.2.2.2 [110/21] via 10.200.0.1, 01:46:17, GigabitEthernet0/0
      3.0.0.0/32 is subnetted, 1 subnets
O IA     3.3.3.3 [110/11] via 10.200.0.1, 01:46:52, GigabitEthernet0/0
      4.0.0.0/32 is subnetted, 1 subnets
O IA     4.4.4.4 [110/31] via 10.200.0.1, 01:46:12, GigabitEthernet0/0
      5.0.0.0/32 is subnetted, 1 subnets
O IA     5.5.5.5 [110/31] via 10.200.0.1, 01:27:27, GigabitEthernet0/0
      9.0.0.0/32 is subnetted, 1 subnets
C        9.9.9.9 is directly connected, Loopback0
      10.0.0.0/8 is variably subnetted, 7 subnets, 4 masks
O IA     10.0.1.0/24 [110/30] via 10.200.0.1, 01:46:18, GigabitEthernet0/0
O IA     10.10.1.0/29 [110/20] via 10.200.0.1, 01:46:53, GigabitEthernet0/0
O IA     10.20.1.0/30 [110/20] via 10.200.0.1, 01:46:53, GigabitEthernet0/0
C        10.200.0.0/30 is directly connected, GigabitEthernet0/0
L        10.200.0.2/32 is directly connected, GigabitEthernet0/0
C        10.200.1.0/24 is directly connected, GigabitEthernet2/0
L        10.200.1.1/32 is directly connected, GigabitEthernet2/0
      192.168.1.0/30 is subnetted, 2 subnets
O IA     192.168.1.0 [110/30] via 10.200.0.1, 01:46:13, GigabitEthernet0/0
O IA     192.168.1.4 [110/30] via 10.200.0.1, 01:46:13, GigabitEthernet0/0
O IA  192.168.2.0/24 [110/40] via 10.200.0.1, 01:46:13, GigabitEthernet0/0
```