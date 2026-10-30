### Phase 7: Database path and ICMP

1. - [X] From each web server, a TCP probe to port **5432** on db-server (by name, once DNS is in) gets a response.
2. - [X] The same probe from R5, and from R1, gets no response.
3. - [X] SSH from web-server-1 to web-server-2 works. The database rule does not cover port 22.
4. - [X] Echo-request from R5 to each server is dropped. Echo-request from R4 to each server succeeds. A non-echo ICMP exchange from R5 to a server still succeeds. You choose a type you can generate (for example an unreachable, or a packet that elicits one) and record which type you used.

**Theory questions.**

- The accepting rule names destination port 5432 and the two web-server addresses. The web server uses an ephemeral source port. Why does a stateful filter still allow the return packets?
  - Because Conntrack registered the established connection.
- Dropping every ICMP packet from R5 would also satisfy the echo-request test. Which useful message would you destroy by doing that, and where would you notice it?
  - We might block ICMP error messages necessary for IP to work.
- web-server-2 reaches db-server by name. List the earlier phases that this one session depends on, in order. A gap in your list is a gap in the design.
  - Phase 1, then 2, then 3, then 4, then 5. Phase 6 and Phase 8 are not on this path.
    1. Phase 1. R4, R2 and SW1 are addressed, so web-server-2 and db-server have a switched path and a gateway.
    2. Phase 2. OSPF adjacencies exist, including Area 1 between R4 and R2 and Area 0 across R2.
    3. Phase 3. R4 has a route to `10.0.1.0/24`, and db-server has a way back to `192.168.2.0/24` (its DHCP default gateway, plus OSPF on R2).
    4. Phase 4. Both servers hold the leased addresses and the DNS resolver `10.10.1.1`.
    5. Phase 5. `db-server.lab.local` resolves to `10.0.1.6`. Without that record the session can still be opened by address, which is not what this test asks for.
    The firewall that allows TCP/5432 is configured in this phase, so it is part of the test rather than an earlier phase. Outbound NAT is not involved: both addresses are inside OSPF.

**Prove it.** The probe results (both web servers succeed; R5 and R1 fail), the echo-request results (R5 fails, R4 succeeds), and the non-echo ICMP result from R5.


Rules on db-server
```shell
iptables -t filter -A INPUT -i lo -j ACCEPT

iptables -t filter -A INPUT -p icmp --icmp-type \
        echo-request -s 5.5.5.5,192.168.1.2 -j DROP

iptables -t filter -A INPUT -p icmp -j ACCEPT

iptables -t filter -A INPUT -p tcp --dport 22 \
        -m conntrack --ctstate NEW,ESTABLISHED \
        -j ACCEPT

iptables -t filter -A INPUT -p tcp --dport 5432 \
        -m conntrack --ctstate NEW,ESTABLISHED \
        -s 10.200.1.6,192.168.2.6 -j ACCEPT

iptables -t filter -A INPUT -m conntrack \
        --ctstate ESTABLISHED,RELATED -j ACCEPT

iptables -t filter -P INPUT DROP
```

TCP 5432 from web servers to db server
```shell
root@web-server-2:~# hping3 -p 5432 -S db-server.lab.local
HPING db-server.lab.local (enp2s0 10.0.1.6): S set, 40 headers + 0 data bytes
len=40 ip=10.0.1.6 ttl=62 DF id=0 sport=5432 flags=RA seq=0 win=0 rtt=10.9 ms
len=40 ip=10.0.1.6 ttl=62 DF id=0 sport=5432 flags=RA seq=1 win=0 rtt=2.8 ms
len=40 ip=10.0.1.6 ttl=62 DF id=0 sport=5432 flags=RA seq=2 win=0 rtt=2.7 ms


root@web-server-1:~# hping3 -p 5432 -S db-server.lab.local
HPING db-server.lab.local (enp2s0 10.0.1.6): S set, 40 headers + 0 data bytes
len=40 ip=10.0.1.6 ttl=61 DF id=0 sport=5432 flags=RA seq=0 win=0 rtt=82.4 ms
len=40 ip=10.0.1.6 ttl=61 DF id=0 sport=5432 flags=RA seq=1 win=0 rtt=122.3 ms
len=40 ip=10.0.1.6 ttl=61 DF id=0 sport=5432 flags=RA seq=2 win=0 rtt=82.2 ms
```

Iptables rules on servers and ping result from R5
```shell
root@db-server:~# iptables -nvL
Chain INPUT (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    0     0 DROP       icmp --  *      *       5.5.5.5              0.0.0.0/0            icmptype 8
    4   400 DROP       icmp --  *      *       192.168.1.2          0.0.0.0/0            icmptype 8
   13  1252 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW,ESTABLISHED
    0     0 ACCEPT     tcp  --  *      *       10.200.1.6           0.0.0.0/0            tcp dpt:5432 ctstate NEW,ESTABLISHED
    0     0 ACCEPT     tcp  --  *      *       192.168.2.6          0.0.0.0/0            tcp dpt:5432 ctstate NEW,ESTABLISHED
    0     0 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED

root@web-server-2:~# iptables -nvL
Chain INPUT (policy DROP 3 packets, 174 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    0     0 DROP       icmp --  *      *       5.5.5.5              0.0.0.0/0            icmptype 8
    5   500 DROP       icmp --  *      *       192.168.1.2          0.0.0.0/0            icmptype 8
    2   168 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW,ESTABLISHED
   13  3709 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED



R5#ping db-server.lab.local
Translating "db-server.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.6, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)

R5#ping web-server-1.lab.local
Translating "web-server-1.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.200.1.6, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R5#ping web-server-2.lab.local
Translating "web-server-2.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.6, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)

```

SSH from web servers to db-server
```shell
root@web-server-1:~# ssh root@db-server.lab.local
The authenticity of host 'db-server.lab.local (10.0.1.6)' can't be established.
ED25519 key fingerprint is SHA256:yhHymt5+CzeO6u1knmNEN9OfOsamaBwox8xtHlTaqRU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? 

root@web-server-2:~# ssh root@db-server.lab.local
The authenticity of host 'db-server.lab.local (10.0.1.6)' can't be established.
ED25519 key fingerprint is SHA256:yhHymt5+CzeO6u1knmNEN9OfOsamaBwox8xtHlTaqRU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? 
```

Successful ping from R4 to all servers
```shell
R4#ping web-server-1.lab.local
Translating "web-server-1.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.200.1.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 83/103/121 ms
R4#ping web-server-2.lab.local
Translating "web-server-2.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R4#ping db-server.lab.local   
Translating "db-server.lab.local"...domain server (10.10.1.1) [OK]
i
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.6, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/7/15 ms

```

Non-ICMP connection from R5 to db-server
```shell
R5#ssh -l root db-server.lab.local 
Translating "db-server.lab.local"...domain server (10.10.1.1) [OK]
i
[Connection to db-server.lab.local aborted: error status 0]
R5#
*Oct 30 17:23:33.456: %SSH-3-NO_MATCH: No matching kex algorithm found: client diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1 server mlkem768x25519-sha256,sntrup761x25519-sha512,sntrup761x25519-sha512@openssh.com,curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,ext-info-s,kex-strict-s-v00@openssh.com 
```