---
title: "NAT"
layout: page
category: basics
order: 5
---

## NAT concepts
* __Inside local address:__ Private IP address
* __Inside global address:__ Public IP address
* __Outside global address:__ The IP address assigned to a host that resides in the outside network
* __Outside local address:__ outside private address when NAT is applied to outside hosts, which is not typical
* __Static NAT:__ Maps one private IP address to a public IP address. This means you're constrained by the number of public IP addresses you have.
* __Dynamic NAT:__ Similar to static NAT, except that IP addresses are assigned dynamically based on what's available in the pool.
* __NAT overload:__ Enables a router to support many devices with few IP addresses by combining IP addresses with port numbers to establish unique connections (sockets).

## NAT troubleshooting
* __Dynamic NAT:__ Symptoms of not having enough addresses include a growing value in the second misses counter in the show ip nat statistics command output, as well as seeing all the pool addresses already in the NAT table.
* IOS processes ACLs before NAT for packets entering an interface, and after translating the addresses for packets exiting an interface.
* The NAT function on one router can be impacted by a routing problem that occurs on another router. Routers outside the network need to be able to route packets to the inside global IP addresses configured on the NAT router.

## Static NAT configuration
The `static` keyword results in a static entry that is never removed from the NAT table due to timeout

    !-- Inside interface
    
    interface GigabitEthernet0/0
      ip address 192.168.1.1 255.255.255.0
      ip nat inside
    
    !-- Outside interface
    
    interface Serial0/0
      ip address 11.1.1.1 255.255.255.0
      ip nat outside
      exit
    
    ip nat inside source static 192.168.1.2 11.1.1.3
    ip nat inside source static 192.168.1.3 11.1.1.4
    ip nat inside source static 192.168.1.4 11.1.1.5

    !-- Verification

    show ip nat translations
    show ip nat statistics

## Dynamic NAT configuration
Dynamic NAT uses an ACL to identify which inside local IP addresses need to be translated, and defines a pool of registered public IP addresses to allocate.

interface GigabitEthernet0/0 
  ip address 10.1.1.3 255.255.255.0 
  ip nat inside

interface Serial0/0/0 
  ip address 200.1.1.251 255.255.255.0 
  ip nat outside
  exit

!-- First/last IP addresses in the pool plus subnet mask

ip nat pool OUTSIDE_ADDR 200.1.1.1 200.1.1.2 netmask 255.255.255.252

!-- ACL for inside IP addresses

access-list 1 permit 10.1.1.0 0.0.0.255

!-- Source ACL containing inside IP addresses to be translated

ip nat inside source list 1 pool OUTSIDE_ADDR

## NAT overload (PAT) configuration

int e0/0
  ip address 192.168.1.1 255.255.255.0
  ip nat inside
int e0/1
  ip address 149.1.252.2 255.255.255.252
  ip nat outside
  exit
!
access-list 1 permit 192.168.1.0 0.0.0.255
!
ip nat inside source list 1 int e0/1 overload
