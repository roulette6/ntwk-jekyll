---
title: "IPv6"
layout: page
category: basics
order: 8
---

IPv6 prefixes are assigned by geographic region -> ISP -> customer to reduce the size of routing tables. E.g., Europe has a single route to the USA.

| Term | Assignment | Example |
| :--- | :--- | :--- |
| Registry prefix | by IANA to a Regional Internet Registry (RIR) | 2340:: /12 |
| ISP prefix | By an RIR to an ISP | 2340:1111 /32 |
| Site prefix or global routing prefix | By an ISP or registry to a customer/site | 2340:1111:AAAA /48 |
| Subnet prefix | By an enterprise engineer for each link | 2340:1111:AAAA:0001 /64 |


## Conventions for writing IPv6 prefixes
* IPv6 has classless addressing with no concept of classful addressing. IPv4 has classful addressing with network bits, subnet bits, and and host bits as well as classless addressing with the prefix bits and the host bits.
* For IPv6, if the prefix length isn't a multiple of 16, the boundary between the __prefix__ and the __interface ID__ parts is inside a quartet. An address with a /56 prefix would have two trailing zeroes in the quartet that's half network and half host bits like this: 2000:1234:5678:9A00::/56
* IPv6 standards reserve the range inside the 2000::/3 prefix as global unicast addresses. This includes addresses beginning with binary 001, which is a 2 (0010) or 3 (0011).
* __Global routing prefix:__ The prefix assigned to a company. Also called a __site prefix__.

## IPv6 global unicast addresses assignment
IPv6 has four major options for IPv6 global unicast address assignment. They are stateful DHCP, stateless autoconfig, static configuration, and static configuration with EUI-64.

### Stateful DHCP for IPv6
This works like IPv4 DHCP, except that the host's first request is multicast and the DHCP server doesn't provide the default router. That's learned though NDP. The name and format of the DHCP messages is different for IPv6. DHCPv6 can be stateful like IPv4--meaning the server remembers assigned IP addresses and their lease--or stateless.

IPv6 multicast addresses have a prefix of __FF00::/8__.

### Stateless autoconfiguration
Stateless autoconfiguration allows hosts to automatically learn key pieces of addressing information such as the prefix and its length and DNS servers. Stateless autoconfiguration uses the following steps:

1. The host uses NDP (especially RS and RA) to learn the prefix, prefix length, and default router.
2. The host uses EUI-64 to derive the interface ID portion of its address.
3. Stateless DHCP to learn the DNS server IP addresses.

### Learning the prefix/length and default router with NDP RA
IPv6's NDP uses ICMPv6 messages called Router Solicitations and Router Advertisements to multicast a request to all routers on the link asking whether they're willing to act as a default gateway and for all the known IPv6 prefixes on that link. For routers to respond to RS messages, they must have at least one IPv6 IP address configured on that link and be configured for IPv6 routing (via the `ipv6 unicast-routing` command). A single router's RA could include the IPv6 addresses and prefixes advertised by other routers on the link.

* RS: Sent to __FF02::02__; means "all routers on this link."
* RA: Sent to __FF02:01__; means "all IPv6 nodes on this link."

### Calculating the interface ID using EUI-64
The interface ID part of an IPv6 address can be set to anything that's not already in use by another host. To guarantee a unique IPv6 address, IPv6 defines the EUI-64 process of deriving an IP address using the host's burned in MAC address, which is itself unique. EUI-64 expands the MAC address into 64 bits by splitting the MAC address into two halves and inserting FFFE in the middle. The seventh bit (reading left-to-right) is then flipped to make the IPv6 address confirm to the EUI-64 format. Example below.

    0034:5678:9ABC --> 0234:56FF:FE78:9ABC
    00000000 -> 00000010, giving 2 instead of 0 as the second digit.
    
Now that the host has derived its IPv6 address and learned its default gateway, it´s time to learn the DNS server IPv6 address.

### Finding the DNS IP addresses using stateless DHCP
In IPv4, DHCP servers offer the same IP address to the same client before it expires and adds it back to the pool if no response is heard from a DHCP client in time to renew the lease. The stateless DHCP server provides clients with the DNS servers' IPv6 address.

### Static IPv6 address configuration
When configuring the IPv6 address statically, you can either configure the entire 128-bit IPv6 address or cofigure the 64-bit prefix and tell the device to use an EUI-64 calculaton for the interface ID portion of the address. A router might still use stateless DHCP to learn the DNS IP addresses when using static IPv6 address configuration.

## Survey of IPv6 addressing

### Overview of IPv6 addressing

### Major differences between IPv4 and IPv6

### Unicast IPv6 addressing

#### Unique local IPv6 addresses

#### Link-local unicast addresses

#### IPv6 unicast address summary

### Multicast and other special IPv6 addresses

### Layer 2 address mapping and DAD

## Configuring IPv6 addresses on Cisco routers

    conf t
    ipv6 unicast-routing               !-- global
    ipv6 cef                           !-- global enable Cisco Express Forwarding
    ipv6 flowset                       !-- global conf flow-label marking
     
    interface <if>
      ipv6 address <addr/len>          !-- static config of entire unicast addr
      ipv6 address <prefix/len> eui-64 !-- static config of 64-bit prefix
      ipv6 address autoconfig          !-- use stateless autoconfig to get an addr
      ipv6 address dhcp                !-- use stateful DHCP
      ipv6 unnumbered <if>             !-- use same IPv6 unicast addr as a ref if
      ipv6 enable                      !-- results in link-local addr
      ipv6 address <addr> link local   !-- must conform to the FE80::/10 prefix
      ipv6 address <addr/len> anycast  !-- designates addr as anycast

### Configuring Static IPv6 Addresses on Routers

    interface et0/1
      ipv6 address 2000:0:0:2::1/64
    
    interface et0/0
      ipv6 address 2000:0:0:1::/64 eui-64

### Multicast groups joined by IPv6 router interfaces

    show ipv6 interface fa0/0
    FastEthernet0/0 is up, line protocol is up
      IPv6 is enabled, link-local address is FE80::213:19FF:FE7B:5004
      No Virtual link-local address(es):
      Global unicast address(es):
        2000::4:213:19FF:FE7B:5004, subnet is 2000:0:0:4::/64 [EUI]
      Joined group address(es):
        FF02::1
        FF02::2
        FF02::1:FF7B:5004
      MTU is 1500 bytes

The output above shows the multicast groups joined by the interface.

* FF02::1: all IPv6 devices
* FF02::2: all IPv6 routers
* FF02::1:FF7B:5004: solicited node multicast addr

### Connected Routes and Neighbors

    show ipv6 route
    IPv6 Routing Table - Default - 7 entries
    Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
            B - BGP, M - MIPv6, R - RIP, I1 - ISIS L1
            I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary, D - EIGRP
            EX - EIGRP external
            O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
            ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
    C   2000:0:0:1::/64 [0/0]
         via Serial0/0/1, directly connected
    L   2000::1:213:19FF:FE7B:5004/128 [0/0]
         via Serial0/0/1, receive
    C   2000:0:0:2::/64 [0/0]
         via FastEthernet0/1, directly connected
    L   2000:0:0:2::2/128 [0/0]
         via FastEthernet0/1, receive
    C   2000:0:0:4::/64 [0/0]
         via FastEthernet0/0, directly connected
    L   2000::4:213:19FF:FE7B:5004/128 [0/0]
         via FastEthernet0/0, receive
    L   FF00::/8 [0/0]
         via Null0, receive

The table output above shows an IPv6 routing table without any dynamic or static routes.

* The connected routes are for any unicast IPv6 addresses on the interface with more than link-local scope.
* The local routes are host routes for the router's unicast IPv6 addresses. They allow the router to more efficiently process packets directed to the router itself as opposed to connected subnets.

### The IPv6 Neighbor Table
The IPv6 neighbor table replaces the IPv4 ARP table, listing the MAC address of devices on the same link.

    debug IPv6 nd              !-- Neighbor Discovery packets
    
    show ipv6 neighbors
    IPv6 Address               Age Link-layer Addr  State  Interface
    2000:0:0:2::3              0   0013.197b.6588   REACH  Fa0/1
    FE80::213:19FF:FE7B:6588   0   0013.197b.6588   REACH  Fa0/1

### Stateless Autoconfiguration

    conf t
    int fa0/1
      no ipv6 address         !-- rem all IPv6 addr; disables IPv6
      ipv6 address autoconfig !-- use stateless autoconfig
    end
    
    show ipv6 router          !-- cached content of received RA msgs
    R2# show ipv6 router
    Router FE80::213:19FF:FE7B:6588 on FastEthernet0/1, last update 0 min
      Hops 64, Lifetime 1800 sec, AddrFlag=0, OtherFlag=0, MTU=1500
      HomeAgentFlag=0, Preference=Medium
      Reachable time 0 (unspecified), Retransmit time 0 (unspecified)
    Prefix 2000:0:0:2::/64 onlink autoconfig
      Valid lifetime 2592000, preferred lifetime 604800

