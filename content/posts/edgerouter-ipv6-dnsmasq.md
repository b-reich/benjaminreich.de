+++ 
draft = true
date = 2021-06-12T11:14:59+02:00
title = "EdgeRouter resolving link-local addresses (IPv6)"
slug = "EdgeRouter resolving link-local addresses (IPv6)"
authors = ["Benjamin Reich"]
tags = ["IPv6", "Ubiquiti", "Edgerouter"]
categories = ["networking"]
+++
The following blogpost will help you to configure the EdgeRouter DHCPv6 server (dnsmasq) to advertise it's link-local address as DNS server to clients.

If you use Dnsmasq for the integration of name resolution for local hostnames. You may want this behavior with the link-local addresses (IPv6).

You can configure Dnsmasq with the the ra-nmes option to solve this issue.

The following config example may help you to configure it. 

```plaintext
configure
set service dns forwarding options 'dhcp-range=::,constructor:switch0,ra-names'
commit; save; exit

sudo service dnsmasq restart
```
*Note: Maybe you have to change the interface behind constructor.*