---
title: "GRE"
layout: default
category: routing
order: 35
---

## GRE Tunnel Concepts
__Generic Routing Encapsulation (GRE)__ defines a header for encapsulating IP packets. Two routers work together with matching configs to create a GRE IP tunnel.

### Routing over GRE Tunnels
GRE tunnels exist between two routers and work like a serial link as far as packet forwarding. Instead of using serial links with serial interfaces, routers use virtual __tunnel interfaces__. The routers have IP addresses on the tunnel interfaces in the same subnet. The routers will encapsulate packets inside tunnel headers instead of serial link HDLC headers, and will even have routes that list tunnel interfaces as outgoing interfaces. Routers treat tunnels like any other link with a point-to-point topology: IPv4 addresses in the same subnet, becoming neighbors and exchanging routes over the tunnel, and using the tunnel as the outgoing interface for routes learned over the tunnel.

The above describes point-to-point GRE tunnels, but DMVPN also uses multipoint GRE tunnels, with more than two endpoints in the same tunnel.

### GRE tunnels over the unsecured network
Any IP network would allow a GRE tunnel to exist. The routers agree to send each other packets over the unsecured network. The virtual tunnel interfaces are created with the command interface tunnel <num>. GRE specifies two headers to create a tunnel: Its own header to manager the tunnel itself and a complete 20-byte IP header called the delivery header, which uses IP addresses from the unsecured network. The routers in the Internet use the outer GRE delivery IP header to route the packet. When the GRE packet arrives to the destination router, the destination router extracts the original IP packet by removing the delivery header and the GRE header. If IPsec encryption is also configured, the original packet is encrypted before being encapsulated in both headers and decrypted after being deencapsulated from both headers.

## Configuring GRE tunnels
Routers must have a route to the tunnel interface of the destination they are trying to reach.

    +--------+             +--------+
    |        |             |        |
    |  src   |XXXXX   XXXXX|  src   |
    |        |     X X     |        |
    |        |      X      |        |
    |        |     X X     |        |
    |  dst   |XXXXX   XXXXX|  dst   |
    |        |             |        |
    +--------+             +--------+

    interface tunnel <num>
      ip address <addr> <mask>
      tunnel mode gre ip
      tunnel source <ip addr | if>
      tunnel destination <ip addr | fqdn>

## Verifying a GRE tunnel
The ultimate test of a tunnel is whether it can pass end-user traffic, but you can use some show commands to learn about the status before trying a ping or traceroute.

    show ip interface brief         !-- shows tunnel
    show interfaces tunnel <num>    !-- status, counters, config
    show ip route <ip addr>         !-- is tunnel an outgoing if on ntwk?
    show run interface tunnel <num>
    traceroute                      !-- by itself, then specify options

## Troubleshooting GRE tunnels

### Tunnel interfaces and interface state
The first thing you should check is that both routers refer to the correct IP addresses in the unsecure part of the network referenced by the `tunnel source` and `tunnel destination` commands. For the tunnel to work, the tunnel interfaces on both routers must be up/up, which requires the `tunnel source` and `tunnel destination` commands.

If the `tunnel source` command references an interface, it must have an IP address and be up/up. If the command references an IP address, it must be assigned to an interface on the local router and the interface must be up/up. For a `tunnel destination` command referencing a destination IP address, the router must have a matching route or else the tunnel interface won't reach up/up state. The matching route may be the default route. If the command references a hostname and it doesn't resolve immediately, IOS will reject the command. If the hostname resolves, IOS will store the IP address instead.

### Layer 3 issues for tunnel interfaces
Two tests to determine whether the tunnel is working:

* Ping the private IP address on the other end of the tunnel. Both router's tunnels must be up/up.
* Enable a routing protocol on both ends of the tunnel. If a neighbor relationship is formed, this proves that packets can flow through the tunnel.

### Issues with ACLs and security
GRE traffic can be filtered by ACLs in devices under and not under the control of the enterprise. Outbound ACLs on a router don't filter packets created by the same router, including new packets created by a GRE tunnel. Where you should start is by looking at inbound ACLs on the tunnel routers, making sure GRE traffic is permitted by those ACLs and any others in the network.

GRE isn't TCP or UDP, but rather another transport protocol indicated in the __Protocol Type__ field of the IP header.

For an ACL to match GRE with a permit command, it needs to have a permit ip command that matches the GRE tunnel's unsecured public addresses or have a permit gre command that also matches the GRE tunnel's unsecured IP addresses.

## mGRE
Imagine one hub and multiple spokes.

### mGRE with NHRP on hub and static on spokes

    !-- hub

    interface fa0/0
      ip address 172.16.1.1 255.255.255.0
      no shut
    !
    interface tunnel0
      ip address 192.168.1.1 255.255.255.0
      no ip redirects
      ip nhrp map multicast dynamic
      ip nhrp network-id 1
      ip nhrp registration timeout 30
      ip nhrp holdtime 600
      no ip split-horizon eigrp 10
      tunnel source 172.16.1.1
      tunnel mode gre multipoint


    !-- spoke-1

    interface fa0/0
      ip address 172.16.1.2 255.255.255.0
      no shut
    !
    interface tunnel0
      ip address 192.168.1.2 255.255.255.0
      ip nhrp network-id 1
      ip nhrp map multicast 172.16.1.1
      ip nhrp map 192.168.1.1 172.16.1.1
      ip nhrp nhs 192.168.1.1
      ip nhrp registration timeout 30
      ip nhrp holdtime 600
      tunnel source 172.16.1.2
      tunnel destination 172.16.1.1


    !-- spoke-2

    interface fa0/0
      ip address 172.16.1.3 255.255.255.0
      no shut
    !
    interface tunnel0
      ip address 192.168.1.3 255.255.255.0
      ip nhrp network-id 1
      ip nhrp map multicast 172.16.1.1
      ip nhrp map 192.168.1.1 172.16.1.1
      ip nhrp nhs 192.168.1.1
      ip nhrp registration timeout 30
      ip nhrp holdtime 600
      tunnel source 172.16.1.3
      tunnel destination 172.16.1.1


### mGRE on hub and spokes

    !-- hub

    interface fa0/0
      ip address 172.16.1.1 255.255.255.0
      no shut
    !
    interface tunnel0
      ip address 192.168.1.1 255.255.255.0
      no ip redirects
      ip nhrp map multicast dynamic
      ip nhrp network-id 1
      ip nhrp registration timeout 30
      ip nhrp holdtime 600
      no ip split-horizon eigrp 10
      tunnel source 172.16.1.1
      tunnel mode gre multipoint


    !-- spoke-1

    interface fa0/0
      ip address 172.16.1.2 255.255.255.0
      no shut
    !
    interface tunnel0
      ip address 192.168.1.2 255.255.255.0
      no ip redirects
      ip nhrp network-id 1
      ip nhrp map multicast 172.16.1.1
      ip nhrp map 192.168.1.1 172.16.1.1
      ip nhrp nhs 192.168.1.1
      ip nhrp registration timeout 30
      ip nhrp holdtime 600
      tunnel source 172.16.1.2
      tunnel mode gre multipoint


    !-- spoke-2

    interface fa0/0
      ip address 172.16.1.3 255.255.255.0
      no shut
    !
    interface tunnel0
      ip address 192.168.1.3 255.255.255.0
      no ip redirects
      ip nhrp network-id 1
      ip nhrp map multicast 172.16.1.1
      ip nhrp map 192.168.1.1 172.16.1.1
      ip nhrp nhs 192.168.1.1
      ip nhrp registration timeout 30
      ip nhrp holdtime 600
      tunnel source 172.16.1.3
      tunnel mode gre multipoint