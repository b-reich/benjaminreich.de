+++ 
draft = false
date = 2021-02-22T11:11:50+01:00
title = "Edgerouter IPv6"
description = "Configure IPv6 on a Edgerouter"
slug = "Configure IPv6 on a Edgerouter"
authors = ["Benjamin Reich"]
tags = ["IPv6", "Ubiquiti", "Edgerouter"]
categories = ["networking"]
externalLink = ""
series = []
+++
The following blogpost will help you to set up IPv6 on your Edgerouter.

## Interfaces
Most ISPs allow you to get a prefix via prefix delegation and use it for your subnets as you like. In germany the prefix size of /56 is very common.

In my example eht0 is may WAN interface. switch0 is my LAN interface and switch0.2 and switch0.3 are tagged vlans on my LAN interface.

```plaintext
configure

set interfaces ethernet eth0 dhcpv6-pd no-dns
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0 host-address '::1'
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0 no-dns
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0 prefix-id ':1'
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0 service slaac
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0.2 host-address '::1'
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0.2 no-dns
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0.2 prefix-id ':2'
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0.2 service slaac
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0.3 host-address '::1'
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0.3 no-dns
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0.3 prefix-id ':3'
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0.3 service slaac
set interfaces ethernet eth0 dhcpv6-pd pd 0 prefix-length /56
set interfaces ethernet eth0 ipv6 address autoconf
set interfaces ethernet eth0 ipv6 dup-addr-detect-transmits 1

commit; save; exit
```
The `host-addresses` are for the interfaces on the router. The `prefix-id` is added to the prefix you receive from the ISP in order to obtain a /64 subnet.

If you are not sure about the prefix length you can save the settings and use tcpdump (on the router) and Wireshark (on your computer) to determine the prefix length.
1. start tcpdump `sudo tcpdump -i eth0 -n ip6 -w /tmp/eth0dump.pcap`
2. renew the prefix `renew dhcpv6-pd interface eth0`
3. copy /tmp/eth0dump.pcap to your computer and open it with Wireshark.
![Wireshark IPv6 DHCPv6 prefix length](/images/posts/Wireshark-DHCPv6-PD.png)

## Firewall
You have to create manualy your firewall rules for IPv6.
The following rules allow outbound traffic, their respones and ICPMv6.
```plaintext
configure

set firewall ipv6-name WANv6_IN default-action drop
set firewall ipv6-name WANv6_IN description 'WAN inbound traffic forwarded to LAN'
set firewall ipv6-name WANv6_IN enable-default-log
set firewall ipv6-name WANv6_IN rule 10 action accept
set firewall ipv6-name WANv6_IN rule 10 description 'Allow established/related sessions'
set firewall ipv6-name WANv6_IN rule 10 state established enable
set firewall ipv6-name WANv6_IN rule 10 state related enable
set firewall ipv6-name WANv6_IN rule 20 action drop
set firewall ipv6-name WANv6_IN rule 20 description 'Drop invalid state'
set firewall ipv6-name WANv6_IN rule 20 state invalid enable
set firewall ipv6-name WANv6_IN rule 30 action accept
set firewall ipv6-name WANv6_IN rule 30 description 'Allow ICPMv6'
set firewall ipv6-name WANv6_IN rule 30 protocol icmpv6
set firewall ipv6-name WANv6_LOCAL default-action drop
set firewall ipv6-name WANv6_LOCAL description 'WAN inbound traffic to the router'
set firewall ipv6-name WANv6_LOCAL enable-default-log
set firewall ipv6-name WANv6_LOCAL rule 10 action accept
set firewall ipv6-name WANv6_LOCAL rule 10 description 'Allow established/related sessions'
set firewall ipv6-name WANv6_LOCAL rule 10 state established enable
set firewall ipv6-name WANv6_LOCAL rule 10 state related enable
set firewall ipv6-name WANv6_LOCAL rule 20 action drop
set firewall ipv6-name WANv6_LOCAL rule 20 description 'Drop invalid state'
set firewall ipv6-name WANv6_LOCAL rule 20 state invalid enable
set firewall ipv6-name WANv6_LOCAL rule 30 action accept
set firewall ipv6-name WANv6_LOCAL rule 30 description 'Allow IPv6 icmp'
set firewall ipv6-name WANv6_LOCAL rule 30 protocol ipv6-icmp
set firewall ipv6-name WANv6_LOCAL rule 40 action accept
set firewall ipv6-name WANv6_LOCAL rule 40 description 'Allow DHCPv6'
set firewall ipv6-name WANv6_LOCAL rule 40 destination port 546
set firewall ipv6-name WANv6_LOCAL rule 40 protocol udp
set firewall ipv6-name WANv6_LOCAL rule 40 source port 547
set firewall ipv6-receive-redirects disable
set firewall ipv6-src-route disable

set interfaces ethernet eth0 firewall in ipv6-name WANv6_IN
set interfaces ethernet eth0 firewall local ipv6-name WANv6_LOCAL

commit; save; exit
```

## Finish
Run `show interfaces` on the router. Now you should see the IPv6 addresses on the LAN interfaces.
```plaintext
Codes: S - State, L - Link, u - Up, D - Down, A - Admin Down
Interface    IP Address                        S/L  Description
---------    ----------                        ---  -----------
eth0         94.126.37.192/22                  u/u  Internet
             2a01:71a0:8000:1:0:d:0:1e/128
             2a01:71a0:8000:1:618:d6ff:fec3:cc14/64
eth1         -                                 u/D  Local
eth2         -                                 u/u  Local 2
eth3         -                                 u/D  Local 2
eth4         -                                 u/D  Local 2
lo           127.0.0.1/8                       u/u
             ::1/128
switch0      192.168.1.1/24                    u/u  Local 2
             2a01:71a0:8010:1e01::1/64
switch0.2    192.168.2.1/24                    u/u  Guest
             2a01:71a0:8010:1e02::1/64
switch0.3    192.168.3.1/24                    u/u  IoT
             2a01:71a0:8010:1e03::1/64
```