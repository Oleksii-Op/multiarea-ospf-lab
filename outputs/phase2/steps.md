### Phase 2: Adjacencies

1. - [X] Run OSPF process 1. Place every interface into the area from section 3. Set reference bandwidth and the authentication from section 4.
2. - [X] Use the network type from the cabling table on the `/29` and on every `/30`.
3. - [X] Choose a network type for the three server LANs (`10.0.1.0/24`, `192.168.2.0/24`, `10.200.1.0/24`) and be ready to defend it. Each of those LANs has one OSPF speaker.
4. - [X] Fill in the prediction columns **before** you inspect the DR election.

| Router | Predicted router ID | Predicted role on 10.10.1.0/29 | Verified role |
|---|---------------------|--------------------------------|---------------|
| R1 | 1.1.1.1             | DROther                        | BDR           |
| R2 | 2.2.2.2             | BDR                            | DROther       |
| R3 | 3.3.3.3             | DR                             | DR            |


After issuing `clear ip ospf process` every role is now as it should be.

**Theory questions.**

- Area 1 contains R2, R4 and R5. Why are the R2–R4 and R2–R5 links still point-to-point?
  - It speeds up the handshake as DR/BDR elections are skipped.
- R2 authenticates Area 0 with MD5 and Area 1 with HMAC-SHA-256 at the same time. Why is that legal on one router? Why does the Area 1 method fail if R4 is a 7200, even with the password `cisco2` typed correctly?
  - In our case Authentication is configured per interface, so it is possible to use different auth methods. Cisco 7200 image does not support HMAC-SHAs Authentication.
- Both ends of R3–R9 use the password `cisco` and the adjacency still refuses to form. Name the authentication parameters, other than the password text, that still have to match.
  - Must match: Timers, the same subnet on adjacent interfaces, IP MTU, area number, Network Type. Routers' ID must be unique.

**Prove it.** The neighbor table from all six routers (every link in section 3 has the neighbor you expect, in the expected state). One interface detail from the `/29` and one from a `/30`, showing network type and authentication. The completed DR table.


R1
```shell
R1(config)#router ospf 1
R1(config-router)#passive-interface g2/0
R1(config-router)#auto-cost reference-bandwidth 10000
% OSPF: Reference bandwidth is changed. 
        Please ensure reference bandwidth is consistent across all routers.
R1(config-router)#network 192.168.122.0 0.0.0.255 area 0
R1(config-router)#network 10.10.1.0 0.0.0.7 area 0  
R1(config-router)#network 1.1.1.1 0.0.0.0 area 0
R1(config)#int g0/0
R1(config-if)#ip ospf authentication message-digest 
R1(config-if)#ip ospf authentication-key cisco
R1(config-if)#ip ospf network broadcast 

R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.122.1
R1(config)#router ospf 1
R1(config-router)#default-information originate always 

R1#show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/BDR        00:00:36    10.10.1.3       GigabitEthernet0/0
3.3.3.3           1   FULL/DR         00:00:33    10.10.1.2       GigabitEthernet0/0

R1#show ip ospf interface g0/0
GigabitEthernet0/0 is up, line protocol is up 
  Internet Address 10.10.1.1/29, Area 0, Attached via Network Statement
  Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State DROTHER, Priority 1
  Designated Router (ID) 3.3.3.3, Interface address 10.10.1.2
  Backup Designated router (ID) 2.2.2.2, Interface address 10.10.1.3
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:01
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 2/2, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 2, Adjacent neighbor count is 2 
    Adjacent with neighbor 2.2.2.2  (Backup Designated Router)
    Adjacent with neighbor 3.3.3.3  (Designated Router)
  Suppress hello for 0 neighbor(s)
  Message digest authentication enabled
      No key configured, using default key id 0

```

R2
```shell
R2(config)#key chain ospf
R2(config-keychain)#key 1
R2(config-keychain-key)#cryptographic-algorithm hmac-sha-256 
R2(config-keychain-key)#key-string cisco2

R2(config)#router ospf 1
R2(config-router)#auto-cost reference-bandwidth 10000
% OSPF: Reference bandwidth is changed. 
        Please ensure reference bandwidth is consistent across all routers.
R2(config-router)#network 10.10.1.0 0.0.0.7 area 0
R2(config-router)#network 10.20.1.0 0.0.0.3 area 0
R2(config-router)#network 10.0.1.0 0.0.0.255 area 0
R2(config-router)#network 192.168.1.4 0.0.0.3 area 1
R2(config-router)#network 192.168.1.0 0.0.0.3 area 1
R2(config-router)#network 2.2.2.2 0.0.0.0 area 0
R2(config-router)#passive-interface g0/0
R2(config)#int g0/2
R2(config-if)#ip ospf network broadcast 
R2(config-if)#ip ospf authentication message-digest  
R2(config-if)#ip ospf authentication-key cisco
R2(config-if)#int range g0/1,g0/3,g1/0
R2(config-if-range)#ip ospf network point-to-point 
R2(config)#int g0/3
R2(config-if)#ip ospf network point-to-point 
R2(config-if)#ip ospf authentication key-chain ospf
*Oct 30 13:33:41.523: %OSPF-5-ADJCHG: Process 1, Nbr 4.4.4.4 on GigabitEthernet0/3 from LOADING to FULL, Loading Done
R2(config)#int g1/0
R2(config-if)#ip ospf network point-to-point 
R2(config-if)#ip ospf authentication key-chain ospf

R2#show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           0   FULL/  -        00:00:36    10.20.1.2       GigabitEthernet0/1
1.1.1.1           1   FULL/DROTHER    00:00:39    10.10.1.1       GigabitEthernet0/2
3.3.3.3           1   FULL/DR         00:00:36    10.10.1.2       GigabitEthernet0/2
5.5.5.5           0   FULL/  -        00:00:33    192.168.1.2     GigabitEthernet1/0
4.4.4.4           0   FULL/  -        00:00:32    192.168.1.6     GigabitEthernet0/3

R2#show ip ospf interface g0/3
GigabitEthernet0/3 is up, line protocol is up (connected) 
  Internet Address 192.168.1.5/30, Area 1, Attached via Network Statement
  Process ID 1, Router ID 2.2.2.2, Network Type POINT_TO_POINT, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:02
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/4, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 12
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 4.4.4.4
  Suppress hello for 0 neighbor(s)
  Cryptographic authentication enabled
    Sending SA: Key 1, Algorithm HMAC-SHA-256 - key chain ospf

R2#show ip ospf interface g0/2
GigabitEthernet0/2 is up, line protocol is up (connected) 
  Internet Address 10.10.1.3/29, Area 0, Attached via Network Statement
  Process ID 1, Router ID 2.2.2.2, Network Type BROADCAST, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State BDR, Priority 1
  Designated Router (ID) 3.3.3.3, Interface address 10.10.1.2
  Backup Designated router (ID) 2.2.2.2, Interface address 10.10.1.3
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:09
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 6
  Last flood scan time is 0 msec, maximum is 1 msec
  Neighbor Count is 2, Adjacent neighbor count is 2 
    Adjacent with neighbor 1.1.1.1
    Adjacent with neighbor 3.3.3.3  (Designated Router)
  Suppress hello for 0 neighbor(s)
  Cryptographic authentication enabled   <- a bug in VIOS
      No key configured, using default key id 0

```

R3
```shell
R3(config)#router ospf 1
R3(config-router)#auto-cost reference-bandwidth 10000
% OSPF: Reference bandwidth is changed. 
        Please ensure reference bandwidth is consistent across all routers.
R3(config-router)#network 10.10.1.0 0.0.0.7 area 0
R3(config-router)#network 10.20.1.0 0.0.0.3 area 0
R3(config-router)#network 10.200.0.0 0.0.0.3 area 2
R3(config-router)#network 3.3.3.3 0.0.0.0 area 0
R3(config)#int g2/0
R3(config-if)#ip ospf network broadcast 
R3(config-if)#ip ospf authentication 
R3(config-if)#ip ospf authentication message-digest 
R3(config-if)#ip ospf authentication-key cisco
*Oct 30 13:10:59.127: %OSPF-5-ADJCHG: Process 1, Nbr 1.1.1.1 on GigabitEthernet2/0 from LOADING to FULL, Loading Done
*Oct 30 13:10:59.131: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet2/0 from LOADING to FULL, Loading Done
R3(config-if)#int g3/0
R3(config-if)#ip ospf authentication message-digest 
R3(config-if)#ip ospf authentication-key cisco
*Oct 30 13:11:29.107: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet3/0 from LOADING to FULL, Loading Done
R3(config-if)#ip ospf network point-to-point 
*Oct 30 13:11:51.783: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet3/0 from FULL to DOWN, Neighbor Down: Interface down or detached
*Oct 30 13:11:51.855: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet3/0 from LOADING to FULL, Loading Done
R3(config-if)#int g0/0
R3(config-if)#ip ospf network point-to-point 
R3(config-if)#ip ospf authentication message-digest 
R3(config-if)#ip ospf authentication-key cisco

R3#show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/  -        00:00:32    10.20.1.1       GigabitEthernet3/0
1.1.1.1           1   FULL/DROTHER    00:00:34    10.10.1.1       GigabitEthernet2/0
2.2.2.2           1   FULL/BDR        00:00:32    10.10.1.3       GigabitEthernet2/0
9.9.9.9           0   FULL/  -        00:00:31    10.200.0.2      GigabitEthernet0/0

R3#show ip ospf interface g0/0
GigabitEthernet0/0 is up, line protocol is up 
  Internet Address 10.200.0.1/30, Area 2, Attached via Network Statement
  Process ID 1, Router ID 3.3.3.3, Network Type POINT_TO_POINT, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:04
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/3, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 3, maximum is 11
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 9.9.9.9
  Suppress hello for 0 neighbor(s)
  Message digest authentication enabled
      No key configured, using default key id 0

R3#show ip ospf interface g2/0
GigabitEthernet2/0 is up, line protocol is up 
  Internet Address 10.10.1.2/29, Area 0, Attached via Network Statement
  Process ID 1, Router ID 3.3.3.3, Network Type BROADCAST, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 3.3.3.3, Interface address 10.10.1.2
  Backup Designated router (ID) 2.2.2.2, Interface address 10.10.1.3
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:05
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 4
  Last flood scan time is 0 msec, maximum is 4 msec
  Neighbor Count is 2, Adjacent neighbor count is 2 
    Adjacent with neighbor 1.1.1.1
    Adjacent with neighbor 2.2.2.2  (Backup Designated Router)
  Suppress hello for 0 neighbor(s)
  Message digest authentication enabled
      No key configured, using default key id 0

```

R9
```shell
R9(config)#router ospf 1
R9(config-router)#passive-interface g2/0
R9(config-router)#auto-cost reference-bandwidth 10000
% OSPF: Reference bandwidth is changed. 
        Please ensure reference bandwidth is consistent across all routers.
R9(config-router)#network 10.200.0.0 0.0.0.3 area 2
R9(config-router)#network 10.200.1.0 0.0.0.255 area 2
R9(config-router)#network 9.9.9.9 0.0.0.0 area 2
R9(config)#int g0/0
R9(config-if)#ip ospf authentication message-digest 
R9(config-if)#ip ospf authentication-key cisco
*Oct 30 13:14:51.211: %OSPF-5-ADJCHG: Process 1, Nbr 3.3.3.3 on GigabitEthernet0/0 from LOADING to FULL, Loading Done
R9(config-if)#ip ospf network point-to-point 
*Oct 30 13:15:14.731: %OSPF-5-ADJCHG: Process 1, Nbr 3.3.3.3 on GigabitEthernet0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
*Oct 30 13:15:14.807: %OSPF-5-ADJCHG: Process 1, Nbr 3.3.3.3 on GigabitEthernet0/0 from LOADING to FULL, Loading Done

R9#show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           0   FULL/  -        00:00:34    10.200.0.1      GigabitEthernet0/0

GigabitEthernet0/0 is up, line protocol is up 
  Internet Address 10.200.0.2/30, Area 2, Attached via Network Statement
  Process ID 1, Router ID 9.9.9.9, Network Type POINT_TO_POINT, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:05
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 3.3.3.3
  Suppress hello for 0 neighbor(s)
  Message digest authentication enabled
      No key configured, using default key id 0
```


R4
```shell
R4(config)#key chain ospf
*Oct 30 13:31:17.372: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Dead timer expired
R4(config-keychain)#key 1
R4(config-keychain-key)#cryptographic-algorithm hmac-sha-256
R4(config-keychain-key)#key-string cisco2

R4(config)#router ospf 1
R4(config-router)#auto-cost reference-bandwidth 10000
% OSPF: Reference bandwidth is changed. 
        Please ensure reference bandwidth is consistent across all routers.
R4(config-router)#passive-interface g0/0
R4(config-router)#network 192.168.2.0 0.0.0.255 area 1
R4(config-router)#network 192.168.1.4 0.0.0.3 area 1
R4(config-router)#network 4.4.4.4 255.255.255.255 area 1
R4(config)#int g0/1
R4(config-if)#ip ospf network point-to-point 

R4#show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/  -        00:00:37    192.168.1.5     GigabitEthernet0/1
R4#show ip ospf interface g0/1
GigabitEthernet0/1 is up, line protocol is up (connected) 
  Internet Address 192.168.1.6/30, Area 1, Attached via Network Statement
  Process ID 1, Router ID 4.4.4.4, Network Type POINT_TO_POINT, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:02
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/2/2, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 2.2.2.2
  Suppress hello for 0 neighbor(s)
  Cryptographic authentication enabled
    Sending SA: Key 1, Algorithm HMAC-SHA-256 - key chain ospf
```

R5
```shell
R5(config)#key chain ospf
R5(config-keychain)#key 1
R5(config-keychain-key)#cryptographic-algorithm hmac-sha-256 
R5(config-keychain-key)#key-string cisco2

R5(config)#router ospf 1
R5(config-router)#auto-cost reference-bandwidth 10000
% OSPF: Reference bandwidth is changed. 
        Please ensure reference bandwidth is consistent across all routers.
R5(config-router)#network 192.168.1.0 0.0.0.3 area 1
R5(config-router)#network 5.5.5.5 255.255.255.255 area 1
R5(config)#int g0/0
R5(config-if)#ip ospf network point-to-point 
*Oct 30 13:36:54.059: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet0/0 from 2WAY to DOWN, Neighbor Down: Interface down or detached
*Oct 30 13:36:54.067: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet0/0 from LOADING to FULL,             
R5(config-if)#ip ospf authentication key-chain ospf

R5(config)#do show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/  -        00:00:38    192.168.1.1     GigabitEthernet0/0
R5(config)#do show ip ospf interface g0/0   
GigabitEthernet0/0 is up, line protocol is up (connected) 
  Internet Address 192.168.1.2/30, Area 1, Attached via Network Statement
  Process ID 1, Router ID 5.5.5.5, Network Type POINT_TO_POINT, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:07
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 2.2.2.2
  Suppress hello for 0 neighbor(s)
  Cryptographic authentication enabled
    Sending SA: Key 1, Algorithm HMAC-SHA-256 - key chain ospf
```