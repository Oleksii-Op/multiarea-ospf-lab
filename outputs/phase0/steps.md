### Phase 0: Build and hygiene

1. - [X] Cable the lab to the table in section 3. Set the hostnames from that table.
2. - [X] Configure every loopback in section 3.

`**Theory question.** The router ID is supposed to be the loopback. After the loopback exists, the running OSPF process can still be using a physical address. Why does a new loopback not always take over immediately?`
    - The router ID is chosen once, when the OSPF process first starts, and a running process does not re-evaluate it when a loopback appears later.

R1
```shell
R1(config)#int loopback 0
*Oct 30 11:07:53.783: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R1(config-if)#ip address 1.1.1.1 255.255.255.255
```

R3
```shell
R3(config)#int loopback 0
*Oct 30 11:05:31.315: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R3(config-if)#ip address 3.3.3.3 255.255.255.255

```

R2
```shell
Switch(config)#host
Switch(config)#hostname R2
R2(config)#int loopback 0
R2(config-if)#ip address 
*Oct 30 12:13:16.257: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up2.2.2.2 255.255.255.255

```
R9
```shell
R9(config)#int loopback 0
*Oct 30 11:06:15.715: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
R9(config-if)#ip address 9.9.9.9 255.255.255.255

```

R4
```shell
Switch(config)#host
Switch(config)#hostname R4
R4(config)#int loopback 0
*Oct 30 12:10:57.345: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to    
R4(config-if)#ip address 4.4.4.4 255.255.255.255
```

R5
```shell
Switch(config)#hostname R5
R5(config)#int loopback 0
R5(config-if)#ip address 5.5.5.5 255.255.255.255
```
SW2
```shell
vIOS-L2-01>enable 
vIOS-L2-01#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
vIOS-L2-01(config)#hostname SW2

```
SW1
```shell
vIOS-L2-01>enable 
vIOS-L2-01#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
vIOS-L2-01(config)#hostname SW1

```

Servers
```shell
root@localhost:~# hostnamectl hostname db-server

root@localhost:~# hostnamectl hostname web-server-1

root@localhost:~# hostnamectl hostname web-server-2

```



```shell
R4(config)#ip routing 
R4(config)#int range g0/0 -3, g1/0
R4(config-if-range)#no sw
R4(config-if-range)#no switchport 
R4(config-if-range)#
*Oct 30 12:11:48.303: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
*Oct 30 12:11:48.464: %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up
*Oct 30 12:11:48.626: %LINK-3-UPDOWN: Interface GigabitEthernet0/2, changed state to down
*Oct 30 12:11:48.785: %LINK-3-UPDOWN: Interface GigabitEthernet0/3, changed state to down
*Oct 30 12:11:48.946: %LINK-3-UPDOWN: Interface GigabitEthernet1/0, changed state to down
*Oct 30 12:11:49.303: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
*Oct 30 12:11:49.464: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
*Oct 30 12:11:49.626: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to down
*Oct 30 12:11:49.785: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/3, changed state to down
*Oct 30 12:11:49.946: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0, changed state to downexit
```