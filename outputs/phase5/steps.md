### Phase 5: DNS

1. - [X] On R1, serve `lab.local` and forward unknown names to `8.8.8.8`.
2. - [X] Publish the six router records from section 4.
3. - [X] Publish the three server records from the lease table. Do this only after the leases exist.

**Theory questions.**

- From web-server-1, `R9.lab.local` answers with `9.9.9.9`, and `archive.debian.org` answers with a public address. How does one resolver produce both results?
  - For local domain it keeps its own host mappings, for entries it doesn't know it sends DNS queries to the upstream server (8.8.8.8)
- Why are the server records written from the lease table, rather than chosen in the DHCP pool before the server boots?
  - Because out-of-the-box DHCP binding are unpredictable, we may end up with wrong entry-to-ip mapping.

**Prove it.** From each server, a lookup of `R9.lab.local`, of `db-server.lab.local`, and of one public name. The public lookup must return an address that is not inside `lab.local`.

```shell
R1(config)#ip dns server 
R1(config)#ip name-server 8.8.8.8
R1(config)#ip domain name lab.local
R1(config)#ip host R2.lab.local 2.2.2.2
R1(config)#ip host R1.lab.local 1.1.1.1
R1(config)#ip host R3.lab.local 3.3.3.3
R1(config)#ip host R4.lab.local 4.4.4.4 
R1(config)#ip host R5.lab.local 5.5.5.5
R1(config)#ip host R9.lab.local 9.9.9.9
R1(config)#ip host db-server.lab.local 10.0.1.6
R1(config)#ip host web-server-1.lab.local 10.200.1.6
R1(config)#ip host web-server-2.lab.local 192.168.2.6

*Oct 30 15:52:09.879: %SYS-5-CONFIG_I: Configured from console by console
R1#ping R9.lab.local
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 9.9.9.9, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 64/108/124 ms
R1#ping db-server.lab.local
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/24/32 ms
```

R4
```shell
R4(config)#ip name-server 10.10.1.1
R4#ping R9.lab.local
Translating "R9.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 9.9.9.9, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 85/89/91 ms

R4#ping db-server.lab.local 
Translating "db-server.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/5/9 ms
```

R2
```shell
R2(config)#ip name-server 10.10.1.1
R2#ping R9.lab.local
Translating "R9.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 9.9.9.9, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 51/58/61 ms

R2#ping db-server.lab.local
Translating "db-server.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/7/9 ms
```

R5
```shell
R5(config)#ip name-server 10.10.1.1
R5#ping R9.lab.local
Translating "R9.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 9.9.9.9, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 60/60/61 ms

R5#ping db-server.lab.local
Translating "db-server.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/5/8 ms
```

R9
```shell
R9(config)# ip name-server 10.10.1.1
R9#ping R9.lab.local
Translating "R9.lab.local"...domain server (10.10.1.1) [OK]

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 9.9.9.9, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/8 ms
R9#ping db-server.lab.local
Translating "db-server.lab.local"...domain server (10.10.1.1) [OK]

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 36/55/64 ms

```

R3
```shell
R3(config)#ip name-server 10.10.1.1

R3#ping R9.lab.local
Translating "R9.lab.local"...domain server (10.10.1.1) [OK]

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 9.9.9.9, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 24/29/36 ms

R3#ping db-server.lab.local
Translating "db-server.lab.local"...domain server (10.10.1.1) [OK]

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/26/36 ms
```