### Phase 8: Inbound SSH

1. - [X] From a host on `192.168.122.0/24`, SSH to `192.168.122.221`, `.222` and `.223` lands on db-server, web-server-1 and web-server-2.
2. - [X] The address written in the server's SSH log, or shown in the server's session table, is an address configured on R1. It is not the outside client's address.
3. - [X] TCP/5432 to those same outside addresses does not reach the database filter as an accepted web-server session. The published service is TCP/22.

**Theory questions.**

- R1's own outside address is a DHCP address in `192.168.122.0/24`, and `.221` is a different address in that same subnet. What has to be true on the outside link before a neighbor of R1 can deliver a frame to `.221`?
  - This IP address must be taken by any other host in the subnet.
- The servers already follow a default route to R1, so a reply to the real outside client would still arrive at R1. Why does this lab still require the server to see a source address that belongs to R1?
  - The default route is enough for the reply, so the source rewrite is not what makes SSH succeed. `192.168.122.0/24` is outside OSPF. Leaving that address visible would put an outside client into the server log and into any later firewall decision. The rewrite makes the session look like it came from R1, an address the servers already route and can name.
- You SSHed in through `.221` and the server saw R1 as the source. Why does that same outside client remain unable to open TCP/5432 on db-server? Two different controls are involved. Name both.
  - The host filter is one control: db-server accepts TCP/5432 only from `10.200.1.6` and `192.168.2.6`. R1's address is not in that list, and neither is `192.168.122.1`.
  - The other control is on R1: the static NAT entry publishes TCP/22 only. A packet to `192.168.122.221:5432` is not translated, so it never arrives at db-server.

**Prove it.** An SSH session to each of the three outside addresses, the source address observed on the server, and a failed TCP/5432 attempt through `.221` from the outside host.


```shell
Oct 30 17:42:29 web-server-1 sshd-session[1243]: Connection reset by 192.168.122.1 port 55890 [preauth]
Oct 30 17:08:05 db-server sshd-session[1307]: Connection closed by 10.200.1.6 port 40654 [preauth]
Oct 30 17:42:32 web-server-2 sshd-session[1506]: Connection closed by 192.168.122.1 port 40812 [preauth]
```

```shell
ssh 192.168.122.221 -p 22
The authenticity of host '192.168.122.221 (192.168.122.221)' can't be established.
ED25519 key fingerprint is SHA256:yhHymt5+CzeO6u1knmNEN9OfOsamaBwox8xtHlTaqRU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?

ssh 192.168.122.222 -p 22
The authenticity of host '192.168.122.222 (192.168.122.222)' can't be established.
ED25519 key fingerprint is SHA256:KdSzxMl8xDium5xOIfZv9oH3ADtAJ2xnsq0mQASG/xk.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?

ssh 192.168.122.223 -p 22
The authenticity of host '192.168.122.223 (192.168.122.223)' can't be established.
ED25519 key fingerprint is SHA256:qJcmP6sLGuAzBG/Kaq4CdcEGvV36hyHobktNAC6toHU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? ^C
```

```shell
nmap -sV -p 22,5432 192.168.122.221
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-30 19:50 EET
Nmap scan report for 192.168.122.221
Host is up (0.034s latency).

PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
5432/tcp closed postgresql
```