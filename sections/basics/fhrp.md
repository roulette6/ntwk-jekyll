---
title: "First Hop Redundancy Protocols (FHRP)"
layout: default
category: basics
order: 7
---

<span id="top"></span>First Hop Redundancy Protocols prevent a single router/switch failure from isolating entire VLAN by having two or more routers/switches use a protocol to share a virtual IP address. This virtual IP address is used as the default gateway, and the gateways negotiate which will respond to requests. Below are the three FHRP that can be used with Cisco gear.

* [HSRP](#hsrp)
* [VRRP](#vrrp)
* [GLBP](#glbp)

## Hot Standby Router Protocol (HSRP)<span id="hsrp"></span>
* HSRP is Cisco-proprietary and is described in [IETF RFC 2281](https://tools.ietf.org/html/rfc2281).
* One router is __active__, one router is __standby__, and the other routers __listen__.
* Routers exchange HSRP hello messages to remain aware of the active router and one another.
    - __IP address:__ 224.0.0.2 (all IPv4 routers)
    - __Protocol/port:__ UDP 1985
    - __Timers:__ 3 sec hello / 10 sec holdtime
    - __Virtual MAC address:__ 0000.0c07.acxx, where XX is the HSRP group # in hex.
* An HSRP group can be assigned an arbitrary group number from 0-255.
    - The convention is to make the group number the same as the VLAN ID.
    - Most Catalyst switches support 16 unique HSRP group numbers.
    - If you have over 16 VLANs, you can use the same HSRP group number for all VLANs, since the number is locally significant to the VLAN.
* Load balancing: Requires two different HSRP groups, with a different active router per group. Clients will need to be split among the two vIPs.

### HSRP router election
* HSRP router election is based on the priority value (0-255). The higher priority wins.
* The default priority is 100.
* If the priority is the same for all routers in a group, the router with the highest IP address wins.
* HSRP routers progress through six states so they can listen for other routers in a group.
    1. Initial: HSRP not running
    2. Learn: Router doesn't know the vIP and is waiting to hear from the active router
    3. Listen: Router knows the vIP and is listening for the active and standby
    4. Speak: Router sends hello messages and participates in the active/standby election process
    5. Standby: Router is candidate to become the next active router
    6. Active: Router is forwarding packets sent to the group's virtual MAC address
* Only the standby router monitors the hello messages from the active
* When a standby router assumes the active role, the next-highest priority router in the Listen state will become the standby.
* Timer values can be given in seconds (1-254) or milliseconds (15-999) if the `msec` keyword is used.
    - The `minimum` keyword forces the router to wait 0-3600 seconds before attempting to overthrow an active router with a lower priority.
    - The `reload` keyword forces the router to wait 0-3600 seconds after a reload before overthrowing an active router with a lower priority. This is handy for routing protocol convergence prior to preemption.

            interface <interface>
              standby <group> preempt [delay [minimum <secs>] [reload <secs>]

### HSRP authentication
For MD5 authentication, the string can be up to 64 characters long. You can provide the string or the hash, or you can use a key chain.

    !-- Plain text authentication

    interface <interface>
      standby <group> authentication <1-8 char string>

    !-- MD5 authentication
    
    interface <if>
      standby <group> authentication md5 key-string [0 | 7] <string or hash>
      exit
    !
    key chain <chain name>
      key <key num>
        key-string [0 | 7] <string/hash>
    interface <if>
      standby <group> authentication md5 key-chain <chain name>

### Conceding the election
To avoid having a router remain active with no link to the outside, you can track an interface and decrement the HSRP priority if the interface goes down so another router will become active.

### HSRP example

    interface vlan 50
      ip address 192.168.1.10 255.255.255.0
      standby 1 priority 200
      standby 1 preempt
      standby 1 ip 192.168.1.1
      standby 1 authentication MyKey
    !
      standby 2 priority 100
      standby 2 ip 192.168.1.2
      standby 2 authentication MyKey
      end

    !-- Verification

    show standby vlan 50 brief
    show standby vlan 50

[Back to top](#top)

***

## Virtual Router Redundancy Protocol (VRRP)<span id="vrrp"></span>
* Alternative to HSRP, defined in [IETF RFC 2338](https://tools.ietf.org/html/rfc2338)
    - __VRRP group members:__ 0-255
    - __Router priority:__ 100 default; range 1-254
    - __Virtual MAC address:__ 0000.5e00.01xx, where xx is the VRRP group number
    - __Advertisements:__ 224.0.0..18 (VRRP) IP protocol 112
    - __Timers:__ 1 sec advertisement / 3 &times; advert int + skew time 
* The active router is called the __master router__. The other routers are __backup routers__.
* All VRRP routers preempt by default if their priority is greater. The default preempt delay is 0 sec.

### VRRP example

    interface vlan 5
      vrrp <group> priority <1-254>
      vrrp <group> timers advertise [msec] <interval>
      vrrp <group> timers learn
      no vrrp <group> preempt
      vrrp <group> preempt [delay <seconds>]
      vrrp <group> authentication <string>
      vrrp <group> ip <ip addr> [secondary]
      vrrp <group> track <object-number> [decrement priority]

    !-- Verification

    show vrrp brief

[Back to top](#top)

***

## Gateway Load Balancing Protocol (GLBP)<span id="glbp"></span>
* Cisco proprietary protocol with built in load balancing.
    - __Group numbers:__ 0-1023
    - __Hello messages:__ to 224.0.0.102 (HSRP) on UDP 3222
    - __Virtual MAC address:__ 0007.b4xx.xxyy, where xxxx represents six zero bits and a 10-bit GLBP group number, and yy represents the virtual forwarder number.
    - __Timers:__ 3 sec hello / 10 sec holdtime
        + Hello can range from 1-60 sec or 50-60,000 msec
        + Holdtime can go to 180 sec or 180,000 msec. Should be at least 3 &times; hello
        + Timers are set on the AVG
* A virtual router comprises multiple routers in the same GLBP group. All routers route a portion of the traffic.
* All clients use the same vIP, but the MAC address is different.

### Active virtual gateway
* One router is elected the __active virtual gateway (AVG)__ by having the highest priority value or the highest IP address.
* The AVG answers all ARP requests. It assigns a virtual MAC address to all routers, four being the limit for a group.
* The other routers are called __active virtual forwarders (AVF)__. Other routers in the group are backup or secondary virtual forwarders.
* AVFs must send hellos to every other GLBP peer. Those that don't will be replaced by the AVG.
* A router that's given a new AVF role might already be an AVF with a different MAC address. The AVG maintains two timers to help it resolve this condition.
    - The __redirect timer__ is used to determine when the AVG will stop using the old vMAC in ARP replies. The AVF corresponding to the old vMAC will continue to respond to that vMAC. The redirect timer defaults to 600 seconds and can range from 0-3600 seconds (1 hr).
    - When the __timeout timer__ expires, the old MAC address and AVF using it are flushed from all GLBP peers. Clients must obtain the new virtual MAC address. The timeout timer defaults to 14,400 seconds (4 hrs) and ranges from 700-64,800 seconds (18 hrs).
* GLBP can also track objects to dynamically change the weight to decide which router becomes the AVF for a virtual MAC address in a group. The default is a weight of 100 and the value can range from 0-254. The default decrement value is 10.
* GLBP supports three load balancing methods:<br />`glbp <group> load-balancing {round-robin | weighted | host-dependent}`
    - __Round Robin:__ Even distribution; recommended
    - __Weighted:__ The weighting value determines the proportion of traffic to send to that AVF
    - __Host dependent:__ Each client always gets the same vMAC in ARP replies.

### Enabling GLBP
You must explicitly indicate the IP address on the gateway you wish to make the AVG.
If the AVG fails, an AVF can take over. If an AVF fails, another AVF will respond to its vMAC so clients relying on the old MAC address will have a way out.

    !-- Switch A
    
    interface vlan 50
      ip address 192.168.1.10 255.255.255.0 
      glbp 1 priority 200 
      glbp 1 preempt
      glbp 1 ip 192.168.1.1
    
    !-- Switch B
    
    interface vlan 50
      ip address 192.168.1.11 255.255.255.0 
      glbp 1 priority 150 
      glbp 1 preempt
      glbp 1 ip 192.168.1.1
    
    !-- Switch C
    
    interface vlan 50
      ip address 192.168.1.12 255.255.255.0 
      glbp 1 priority 100 
      glbp 1 ip 192.168.1.1
    
    !-- Verification

    show glbp [brief]

[Back to top](#top)