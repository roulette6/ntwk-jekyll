---
title: "RIP"
layout: default
category: routing
order: 10
---

RIP stands for Routing Information Protocol

## RIPv2
Timers:

* Update (30s): Send complete routing table
* Invalid (180s): Declare route as invalid b/c not received or metric is 16
* Hold-down (180s): Route kept so so all routers in topology to learn that it's unreachable
* Flush (240s): Invalid route removed from routing table
* Garbage-collection timer (120s): Routes that time out are left in the routing table for a short time so neighbors can be notified that the route has been dropped. Garbage collection removes these routes. 

### Configuration

    router rip
      version 2
      network <ntwk id>
      neighbor <ip addr>
      [no] passive interface {<if> | default}
      maximum-paths <num> !-- default 4
      [no] auto-summary
      !
      default-information originate
      redistribute <protocol> <asn | pid>
      default-metric <hops>
      timers basic 1 2 3 4

    interface <if>
      ip rip send version {1 | 2}
      ip rip receive version {1 | 2}
      !
      ip rip authentication key-chain <name>
      ip rip authentication mode {text | md5}
      !
      ip summary-address rip <ntwk> <mask>
      !
      [no] ip split-horizon
     
### Verification
     
    show ip route rip
    show ip protocols
    show ip rip database 

### Troubleshooting
* Missing or incorrect network commands
* Passive interfaces
* Auto-summary


## RIP next generation (RIPng)
* RIPng is similar to RIPv2, with differences being a result of using IPv6.
* RIPng doesn't natively support authentication; it relies on IPsec.
* RIPng uses __UDP 521__ (RIPv2 uses UDP 520)
* Multicast addr: __FF02::9__
* Authentication: IPv6 AH (authentication header)/ESP

### RIPng timers
* __Update:__ How frequently to send entire routing table. Default is every 30 seconds.
* __Timeout:__ Associated with each route, reset whenever an update msg is received for the route. Time until route is considered expired.
* __Garbage collection:__ Associated with each route, the route is removed after it's expired. The buffer exists so router can notify neighbors that route is no longer valid.

### Configuring RIPng

    conf t
    ipv6 unicast-routing
    ipv6 router rip <name>
      maximum-paths <num>
      timers <updt> <rt timeout> <rt holddown> <rt garbage collect>
     
    interface <if>
      ipv6 rip <name> enable   !-- rejected if there's no IPv6 addr

If you forget the `router` command, the interface command will enable the process with the name indicated. Once configured on an interface, RIPng does the following:

1. Sends RIP updates on that interface
2. Starts processing RIP updates received on that interface
3. It advertises the connected routes on that interface (any IPv6 addresses configured on that interface, but not the link-local address or host routes with a /128 prefix).

### Verifying RIPng 
The RIPv2 and RIPng show commands are very similar. The biggest difference is that for IPv4 `show ip protocols` shows more information that for IPv6 is spread among several commands.

RIPng uses the link-local IPv6 address as the next-hop IP address. To see which routers use which link-local addresses, use the command `show cdp entry <name>`, which will show both IPv4 and IPv6 addresses, including link-local.

    show ipv6 route
    show ipv6 route rip
    show ipv6 route <pfx/len>
    show ipv6 protocols         !-- ifs where RIPng enabled
    show ipv6 rip database
    show ipv6 rip               !-- RIP timers
    show ipv6 rip next-hops     !-- list routing info srcs
