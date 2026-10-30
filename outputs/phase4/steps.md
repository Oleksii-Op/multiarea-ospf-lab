### Phase 4: DHCP

1. - [X] Build the three pools from section 4. Lease time is 7 days. The gateway address is excluded.
2. - [X] Bring the servers up as DHCP clients. Record what they received. These addresses are the ones you will put in DNS and in the database filter. A later renewal that changes an address means those two things must be edited to match.

| Host | MAC               | Leased address | Gateway received | DNS received |
|---|-------------------|----------------|------------------|--------------|
| db-server | 0c:66:ce:f4:00:00 | 10.0.1.6       | Yes              | Yes          |
| web-server-1 | 0c:37:cd:79:00:00 | 10.200.1.6               | Yes              | Yes          |
| web-server-2 | 0c:7c:58:c0:00:00 | 192.168.2.6    | Yes              | Yes          |

**Theory questions.**

- The lease is 7 days on a lab you might finish in an afternoon. What breaks in later phases if the lease is five minutes instead?
  - We will have to change DNS host binding and static IP configurations every 5 minutes.
- What does db-server lose if the pool is allowed to offer `10.0.1.1`?
  - It might take the IP address of its default gateway and lose connectivity.
- All three pools sit on the router that owns the LAN. If you moved the pools to R1 and left the servers where they are, which extra function would the server LANs need, and why? Explain it. Do not build it.
    - We would configure dhcp relay agents with `ip-helper`

**Prove it.** The binding table on R2, R4 and R9, and the completed lease table. Each server can ping its own gateway.


R9
```shell
R9(config)#ip dhcp excluded-address 10.200.1.1 10.200.1.5
R9(config)#ip dhcp pool R9_SUBNET
R9(dhcp-config)#lease 7
R9(dhcp-config)#default-router 10.200.1.1
R9(dhcp-config)#dns-server 10.10.1.1
R9(dhcp-config)#domain-name lab.local
R9(dhcp-config)#network 10.200.1.0 /24

R9(config)#do show ip dhcp binding 
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
10.200.1.6          ff5d.e26c.1500.0200.    Nov 06 2026 02:19 PM    Automatic
                    00ab.1171.bd2c.2d50.
                    19d3.38
```

R4
```shell
R4(config)#ip dhcp excluded-address 192.168.2.1 192.168.2.5
R4(config)#ip dhcp pool R4_SUBNET
R4(dhcp-config)#dns-server 10.10.1.1
R4(dhcp-config)#default-router 192.168.2.1
R4(dhcp-config)#lease 5
R4(dhcp-config)#domain-name lab.local
R4(dhcp-config)#network 192.168.2.0 /24

R4(config)#do show ip dhcp binding 
Bindings from all pools not associated with VRF:
IP address      Client-ID/              Lease expiration        Type       State      Interface
                Hardware address/
                User name
192.168.2.6      ff5d.e26c.1500.0200.    Nov 04 2026 03:00 PM    Automatic  Active     GigabitEthernet0/0
                00ab.111a.90bb.6d5b.
                34b7.2f
```

R2
```shell
R2(config)#ip dhcp excluded-address 10.0.1.1 10.0.1.5
R2(config)#ip dhcp pool R2_SUBNET
R2(dhcp-config)#default-router 10.0.1.1
R2(dhcp-config)#dns-server 10.10.1.1
R2(dhcp-config)#lease 7
R2(dhcp-config)#domain-name lab.local
R2(dhcp-config)#network 10.0.1.0 /24

R2(config)#do show ip dhcp binding         
Bindings from all pools not associated with VRF:
IP address      Client-ID/              Lease expiration        Type       State      Interface
                Hardware address/
                User name
10.0.1.6         ff5d.e26c.1500.0200.    Nov 06 2026 03:03 PM    Automatic  Active     GigabitEthernet0/0
                00ab.1130.7e00.9cf1.
                87e1.cf
```