+++ 
draft = true
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

Most ISPs allow you to get a prefix via prefix delegation and use it for your subnets as you like. In germany the prefix size of /56 is very common.

In my example eht0 is may WAN interface. switch0 is my LAN interface and switch0.2 and switch0.3 are tagged vlans on my LAN interface.

```plaintext
set interfaces ethernet eth0 dhcpv6-pd no-dns
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0 host-address '::1'
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0 no-dns
set interfaces ethernet eth0 dhcpv6-pd pd 0 interface switch0 prefix-id '::1'
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
```


```shell
sudo tcpdump -i eth0 -n ip6 -w /tmp/eth0dump.pcap
```