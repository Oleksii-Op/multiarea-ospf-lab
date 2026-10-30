### Phase 1: Link addressing

1. - [X] Address every interface in section 3. Bring the links up.
2. - [X] On R2, R4 and R5 the listed interfaces are routed ports.
3. - [X] From each router, ping the router address at the other end of each router-to-router link. Server addresses do not exist yet; those pings belong to Phase 4.

**Theory question.** SW2 exists so that three routers share `10.10.1.0/29`. Describe what that subnet would look like to OSPF if the three routers were cabled to each other with no switch in the middle. Which design goal of Area 0 would you lose?
 -   The design goal you would lose is Area 0's multi-access segment: one broadcast network where R1, R2, and R3 elect a DR and a BDR.

**Prove it.** The interface address list from each router, and a successful ping across every router-to-router link in section 3.


R1
```shell
R1(config)#int g2/0
R1(config-if)#ip address dhcp
R1(config-if)#no shutdown 
R1(config-if)#int g0/0
R1(config-if)#ip add
*Oct 30 11:04:34.899: %LINK-3-UPDOWN: Interface GigabitEthernet2/0, changed state to up
*Oct 30 11:04:35.899: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet2/0, changed state to up
R1(config-if)#ip address 10.10.1.1 255.255.252.25
*Oct 30 11:04:47.883: %DHCP-6-ADDRESS_ASSIGN: Interface GigabitEthernet2/0 assigned DHCP address 192.168.122.25, mask 255.255.255.0, hostname R1
R1(config-if)#ip address 10.10.1.1 255.255.255.248
R1(config-if)#no shutdown 
*Oct 30 11:05:44.639: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
*Oct 30 11:05:45.639: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up

R1(config)#do show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/0         10.10.1.1       YES NVRAM  up                    up      
Ethernet1/0                unassigned      YES NVRAM  administratively down down    
Ethernet1/1                unassigned      YES NVRAM  administratively down down    
Ethernet1/2                unassigned      YES NVRAM  administratively down down    
Ethernet1/3                unassigned      YES NVRAM  administratively down down    
GigabitEthernet2/0         192.168.122.25  YES DHCP   up                    up      
GigabitEthernet3/0         unassigned      YES NVRAM  administratively down down    
Loopback0                  1.1.1.1         YES NVRAM  up                    up    

R1(config)#do ping 192.168.122.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.122.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/24/32 ms
R1(config)#do ping 10.10.1.2    
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.1.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 20/24/28 ms
R1(config)#do ping 10.10.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/24/32 ms
```

R3
```shell
R3(config)#int g2/0
R3(config-if)#ip address 10.10.1.2 255.255.255.248
R3(config-if)#no shutdown 
R3(config-if)#int g3/0
*Oct 30 11:50:18.307: %LINK-3-UPDOWN: Interface GigabitEthernet2/0, changed state to up
*Oct 30 11:50:19.307: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet2/0, changed state to up
R3(config-if)#int g3/0
R3(config-if)#ip address 10.20.1.2 255.255.255.252
R3(config-if)#no shutdown 
R3(config-if)#int g0/0
*Oct 30 11:50:55.471: %LINK-3-UPDOWN: Interface GigabitEthernet3/0, changed state to up
*Oct 30 11:50:56.471: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet3/0, changed state to up
R3(config-if)#ip address 10.200.0.1 255.255.255.252
*Oct 30 11:51:23.891: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
*Oct 30 11:51:24.891: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up

R3(config)#do show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES unset  administratively down down    
GigabitEthernet0/0         10.200.0.1      YES manual up                    up      
Ethernet1/0                unassigned      YES unset  administratively down down    
Ethernet1/1                unassigned      YES unset  administratively down down    
Ethernet1/2                unassigned      YES unset  administratively down down    
Ethernet1/3                unassigned      YES unset  administratively down down    
GigabitEthernet2/0         10.10.1.2       YES manual up                    up      
GigabitEthernet3/0         10.20.1.2       YES manual up                    up      
Loopback0                  3.3.3.3         YES manual up                    up   

R3(config)#do ping 10.200.0.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.200.0.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 8/24/32 ms
R3(config)#do ping 10.10.1.1 
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/23/28 ms
R3(config)#do ping 10.10.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

```

R9
```shell
R9(config)#int g0/0
R9(config-if)#ip address 10.200.0.2 255.255.255.252
R9(config-if)#no shutdown 
R9(config-if)#int g2/0
*Oct 30 11:52:16.831: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
*Oct 30 11:52:17.831: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
R9(config-if)#ip address 10.200.1.1 255.255.255.0
R9(config-if)#no shutdown 
*Oct 30 11:52:57.203: %LINK-3-UPDOWN: Interface GigabitEthernet2/0, changed state to up
*Oct 30 11:52:58.203: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet2/0, changed state to up

Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/0         10.200.0.2      YES manual up                    up      
Ethernet1/0                unassigned      YES NVRAM  administratively down down    
Ethernet1/1                unassigned      YES NVRAM  administratively down down    
Ethernet1/2                unassigned      YES NVRAM  administratively down down    
Ethernet1/3                unassigned      YES NVRAM  administratively down down    
GigabitEthernet2/0         10.200.1.1      YES manual up                    up      
GigabitEthernet3/0         unassigned      YES NVRAM  administratively down down    
Loopback0                  9.9.9.9         YES NVRAM  up                    up    

R9(config)#do ping 10.200.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.200.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 28/42/60 ms

```

R2
```shell
R2(config)#ip routing 
R2(config)#int range g0/0-3,g1/0
R2(config-if-range)#no switchport 
*Oct 30 12:16:13.180: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
*Oct 30 12:16:13.340: %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up
*Oct 30 12:16:13.528: %LINK-3-UPDOWN: Interface GigabitEthernet0/2, changed state to up
*Oct 30 12:16:13.688: %LINK-3-UPDOWN: Interface GigabitEthernet0/3, changed state to up
*Oct 30 12:16:13.850: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to up
*Oct 30 12:16:14.180: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
*Oct 30 12:16:14.340: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
*Oct 30 12:16:14.528: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to up
*Oct 30 12:16:14.688: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/3, changed state to up
*Oct 30 12:16:14.850: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to up

R2(config)#int g0/0
R2(config-if)#ip address 10.0.1.1 255.255.255.0
R2(config-if)#int g0/2
R2(config-if)#ip address 10.10.1.3 255.255.255.248
R2(config-if)#int g0/3
R2(config-if)#ip address 192.168.1.5 255.255.255.252
R2(config-if)#int g1/0
R2(config-if)#ip address 192.168.1.1 255.255.255.252

R2(config)#do show ip int br                   
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     10.0.1.1        YES manual up                    up      
GigabitEthernet0/1     10.20.1.1       YES manual up                    up      
GigabitEthernet0/2     10.10.1.3       YES manual up                    up      
GigabitEthernet0/3     192.168.1.5     YES manual up                    up      
GigabitEthernet1/0     192.168.1.1     YES manual up                    up      
GigabitEthernet1/1     unassigned      YES unset  down                  down    
GigabitEthernet1/2     unassigned      YES unset  down                  down    
GigabitEthernet1/3     unassigned      YES unset  down                  down    
GigabitEthernet2/0     unassigned      YES unset  down                  down    
GigabitEthernet2/1     unassigned      YES unset  down                  down    
GigabitEthernet2/2     unassigned      YES unset  down                  down    
GigabitEthernet2/3     unassigned      YES unset  down                  down    
Loopback0   

R2(config-if)#do ping 10.20.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.20.1.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 19/27/30 ms
R2(config-if)#do ping 10.10.1.1                   
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 18/28/34 ms

```

R4
```shell
R4(config)#ip routing 
R4(config)#int range g0/0 - 1
R4(config-if-range)#no switchport 

R4(config)#int g0/1  
R4(config-if)#ip address 192.168.1.6 255.255.255.252
R4(config-if)#no shutdown 
R4(config-if)#int g0/0
R4(config-if)#ip address 192.168.2.1 255.255.255.0
R4(config-if)#no shutdown 

R4(config)#do show ip int br
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.2.1     YES manual up                    up      
GigabitEthernet0/1     192.168.1.6     YES manual up                    up      
GigabitEthernet0/2     unassigned      YES unset  down                  down    
GigabitEthernet0/3     unassigned      YES unset  down                  down    
GigabitEthernet1/0     unassigned      YES unset  down                  down    
GigabitEthernet1/1     unassigned      YES unset  down                  down    
GigabitEthernet1/2     unassigned      YES unset  down                  down    
GigabitEthernet1/3     unassigned      YES unset  down                  down    
GigabitEthernet2/0     unassigned      YES unset  down                  down    
GigabitEthernet2/1     unassigned      YES unset  down                  down    
GigabitEthernet2/2     unassigned      YES unset  down                  down    
GigabitEthernet2/3     unassigned      YES unset  down                  down    
Loopback0              4.4.4.4         YES manual up                    up      
R4(config)#do ping 192.168.1.5
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.5, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms

```

R5
```shell

R5(config)#ip routing 
R5(config)#int g0/0
R5(config-if)#no switchport 
*Oct 30 12:22:10.522: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
*Oct 30 12:22:11.522: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
R5(config-if)#ip add
R5(config-if)#ip address 192.168.1.2 255.255.255.252

R5(config)#do show ip int br
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.2     YES manual up                    up      
GigabitEthernet0/1     unassigned      YES unset  down                  down    
GigabitEthernet0/2     unassigned      YES unset  down                  down    
GigabitEthernet0/3     unassigned      YES unset  down                  down    
GigabitEthernet1/0     unassigned      YES unset  down                  down    
GigabitEthernet1/1     unassigned      YES unset  down                  down    
GigabitEthernet1/2     unassigned      YES unset  down                  down    
GigabitEthernet1/3     unassigned      YES unset  down                  down    
GigabitEthernet2/0     unassigned      YES unset  down                  down    
GigabitEthernet2/1     unassigned      YES unset  down                  down    
GigabitEthernet2/2     unassigned      YES unset  down                  down    
GigabitEthernet2/3     unassigned      YES unset  down                  down    
Loopback0              5.5.5.5         YES manual up                    up   

R5(config)#do ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
```