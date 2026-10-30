### Phase 6: Outbound NAT

1. - [X] Translate the three server subnets to the DHCP address on R1 Gi2/0 when they talk to the Internet.
2. Install `openssh-server` and `hping3` on each server from the distribution repositories. The install is the test, not a side task.

**Theory questions.**

- A server resolves `archive.debian.org`, then the package index fails to download. Which of the two functions (DNS forwarding, address translation) is missing, and which table on R1 tells you?
  - Address translation fails. `show ip nat translations` might give a hint that.
- The three servers share one outside address. What does the translation remember so that return traffic comes back to the right server?
  - Source address and source port/Destination address and port
- R1 itself can already reach `8.8.8.8` because Gi2/0 has an upstream gateway from DHCP. Why is that fact not enough for a server on `192.168.2.0/24` to download a package?
  - A gateway that R1 G2/0 is connected to does not know how to reach internal network or might be configured to drop packets from RFC 1918 networks.
**Prove it.** A finished package install on each server, and a translation entry on R1 that shows the server's inside address during the download.



R1
```shell
R1(config)#ip access-list standard NAT_OVERLOAD
R1(config-std-nacl)#1 permit 10.0.1.0 0.0.0.255 
R1(config-std-nacl)#2 permit 192.168.2.0 0.0.0.255
R1(config-std-nacl)#3 permit 10.200.1.0 0.0.0.255
R1(config-std-nacl)#deny any 

R1(config)#int g2/0
R1(config-if)#ip nat outside 
R1(config-if)#int g0/0
*Oct 30 15:57:03.363: %LINEPROTO-5-UPDOWN: Line protocol on Interface NVI0, changed state to up
R1(config-if)#ip nat inside 

R1(config)#ip nat inside source list NAT_OVERLOAD interface g2/0 overload 

R1(config)#do show ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
udp 192.168.122.25:41586 10.0.1.6:41586   216.250.115.174:123 216.250.115.174:123
tcp 192.168.122.25:44790 10.0.1.6:44790   140.248.138.132:443 140.248.138.132:443
udp 192.168.122.25:45427 10.0.1.6:45427   216.250.115.174:123 216.250.115.174:123
udp 192.168.122.25:49617 10.0.1.6:49617   216.250.115.174:123 216.250.115.174:123
tcp 192.168.122.25:59786 10.0.1.6:59786   140.248.138.132:443 140.248.138.132:443
tcp 192.168.122.25:36758 10.200.1.6:36758 140.248.138.132:443 140.248.138.132:443
tcp 192.168.122.25:36770 10.200.1.6:36770 140.248.138.132:443 140.248.138.132:443
udp 192.168.122.25:42142 10.200.1.6:42142 69.89.207.199:123  69.89.207.199:123
tcp 192.168.122.25:45862 10.200.1.6:45862 140.248.138.132:443 140.248.138.132:443
udp 192.168.122.25:52653 10.200.1.6:52653 69.89.207.199:123  69.89.207.199:123
udp 192.168.122.25:55735 10.200.1.6:55735 69.89.207.199:123  69.89.207.199:123
udp 192.168.122.25:36302 192.168.2.6:36302 69.89.207.199:123 69.89.207.199:123
tcp 192.168.122.25:42140 192.168.2.6:42140 140.248.138.132:443 140.248.138.132:443
Pro Inside global      Inside local       Outside local      Outside global
udp 192.168.122.25:53787 192.168.2.6:53787 69.89.207.199:123 69.89.207.199:123
udp 192.168.122.25:59422 192.168.2.6:59422 69.89.207.199:123 69.89.207.199:123

```


Servers
```shell
root@db-server:~# apt install -y openssh-server hping3
Installing:                     
  hping3  openssh-server

Summary:
  Upgrading: 0, Installing: 19, Removing: 0, Not Upgrading: 88
  Download size: 5002 kB
  Space needed: 22.6 MB / 1874 MB available

Get:19 https://deb.debian.org/debian trixie/main amd64 libxmuu1 amd64 2:1.1.3-3+b4 [21.9 kB]
Get:20 https://deb.debian.org/debian trixie/main amd64 xauth amd64 1:1.1.2-1.1 [35.9 kB]
Fetched 5002 kB in 1min 11s (70.8 kB/s)                                        
Preconfiguring packages ...



root@web-server-1:~# apt install -y openssh-server hping3
Installing:                     
  hping3  openssh-server

Summary:
  Upgrading: 0, Installing: 19, Removing: 0, Not Upgrading: 88
  Download size: 5002 kB
  Space needed: 22.6 MB / 1874 MB available

Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Get:2 https://deb.debian.org/debian trixie/main amd64 libedit2 amd64 3.1-20250104-1 [93.8 kB]
Get:3 https://deb.debian.org/debian trixie/main amd64 libcbor0.10 amd64 0.10.2-2 [28.3 kB]

Get:16 https://deb.debian.org/debian trixie/main amd64 libx11-data all 2:1.8.12-1 [343 kB]
Get:17 https://deb.debian.org/debian trixie/main amd64 libx11-6 amd64 2:1.8.12-1 [815 kB]
Get:19 https://deb.debian.org/debian trixie/main amd64 libxmuu1 amd64 2:1.1.3-3+b4 [21.9 kB]
Get:20 https://deb.debian.org/debian trixie/main amd64 xauth amd64 1:1.1.2-1.1 [35.9 kB]
Fetched 5002 kB in 42s (119 kB/s) 
Preconfiguring packages ...


root@web-server-2:~# apt install -y openssh-server hping3
Installing:                     
  hping3  openssh-server

Summary:
  Upgrading: 0, Installing: 19, Removing: 0, Not Upgrading: 88
  Download size: 5002 kB
  Space needed: 22.6 MB / 1874 MB available

Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Get:2 https://deb.debian.org/debian trixie/main amd64 libedit2 amd64 3.1-20250104-1 [93.8 kB]
Get:3 https://deb.debian.org/debian trixie/main amd64 libcbor0.10 amd64 0.10.2-2 [28.3 kB]
Get:15 https://deb.debian.org/debian trixie/main amd64 libxcb1 amd64 1.17.0-2+b1 [144 kB]

Get:19 https://deb.debian.org/debian trixie/main amd64 libxmuu1 amd64 2:1.1.3-3+b4 [21.9 kB]
Get:20 https://deb.debian.org/debian trixie/main amd64 xauth amd64 1:1.1.2-1.1 [35.9 kB]
Fetched 5002 kB in 50s (101 kB/s)                                              
Preconfiguring packages ...

```