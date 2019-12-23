---
title: "IP addresses"
layout: default
category: basics
order: 3
---

IP addresses are Internet Protocol addresses, which correspond to layer three of the OSI model. There are two Internet Protocol versions: IPv4 and IPv6. Typically, when people say "IP," they mean "IPv4."

IP addresses are logical addresses assigned to hosts. An IP address is "logical" and not "physical" because the same device can be given a different IP address, just like you can move from one mailing address to another.

IP networks were originally divided into different classes, which were meant for organizations of varying sizes. These days, however, we [subnet](/subnetting) classful networks into smaller subnetworks to have smaller broadcast domains.

## Public IP address assignment
The Internet Corporation for Assigned Names and Numbers (ICANN) owns the process of IP address allocation. The Internet Assigned Numbers Authority (IANA) carries out ICANN's policies. Together they define which IP addresses can be allocated to which geographic regions, dev of DNS, and TLDs.

* ICANN and IANA group public IP addr by geographic region ->
* IANA allocates them to Regional Internet Registries (RIR) ->
* to National Internet Registries (NIR) / Local Internet Registries (LIR; incl. ISPs) ->
* end-user orgs.

Consecutive IP address ranges are assigned by geography to reduce the number of routes in routing tables from a few tens of millions to a few hundred thousand.


## Classes of IPv4 networks

| Class | First octet value | Hosts | Default mask |
|:--- |:--- |:--- |:--- |
| A | 1-126 | 16,277,214 | 255.0.0.0 |
| B | 128-191 | 65,534 | 255.255.0.0 |
| C | 192-223 | 254 | 255.255.255.0 |
| D | 224-239 | Multicast | N/A |
| E | 240-255 | Experimental | N/A |

## Private addresses
Private addresses are not routable to the Internet. NAT is required to make these addresses routable. These are covered in IETF RFC 1918.

| Class | Addresses | CIDR notation |
|:--- | :--- | :--- |
| A | 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 |
| B | 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 |
| C | 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 |

## Reserved IP addresses

| Value or range | Reason |
| :--- | :--- |
| 0.0.0.0/8 | Self-ID on a local subnet |
| 127.0.0.0/8 | Loopback testing |
| 169.254.0.0/16 | Default IPv4 addr assignment when DHCP fails |
| 192.0.2.0/24 | Reserved for docs and example code |
| 192.88.99.0/24 | IPv6-to-IPv4 relay (6to4 relay; RFC 3068) |
| 198.18.0.0/15 | Benchmark testing for Internet devices (RFC 2544) |
