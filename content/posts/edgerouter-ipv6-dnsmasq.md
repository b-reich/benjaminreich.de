+++ 
draft = true
date = 2021-06-12T11:14:59+02:00
title = "EdgeRouter resolving local IPv6 addresses"
slug = "EdgeRouter resolving local IPv6 addresses"
authors = ["Benjamin Reich"]
tags = ["IPv6", "Ubiquiti", "Edgerouter"]
categories = ["networking"]
+++
The following blogpost will help you to configure the EdgeRouter DHCPv6 server (dnsmasq) to resolve IPv6 and IPv4 addresses.

You can configure Dnsmasq with the the ra-nmes option to solve this issue.

The following config example may help you to configure it. 

```plaintext
configure
set service dhcp-server use-dnsmasq enable 
set service dns forwarding options 'dhcp-range=::,constructor:switch0,ra-names'
commit; save; exit

sudo service dnsmasq restart
```
*Note: Maybe you have to change the interface behind constructor.*