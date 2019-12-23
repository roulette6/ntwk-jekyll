---
title: "OSPF"
layout: page
category: routing
order: 20
---

## <span id="toc"></span>TOC
* [Fundamental OSPF concepts](#fundamentals)
    + [Basic OSPF configuration](#basic-config)
    + [Basic show commands](#basic-show-cmds)
    + [Hello and dead timers](#timers)
    + [OSPF neighbors and adjacencies on WANs](#wan-adjacencies)
    + [OSPF virtual links](#virtual-links)
* [The OSPF link-state database (LSDB)](#lsdb)
    + [Metric tuning](#metric-tuning)
* [Advanced OSPF concepts](#advanced-concepts)
    + [Route filtering](#route-filtering)
        + [Filtering type 3 summary LSAs](#filtering-type3)
        + [Filtering OSPF routes added to the](#filtering-rt-table)
    + [Route summarization](#summarization)
        + [Manual summarization at ASBRs](#asbr-summarization)
    + [Default routes and stubby areas](#default-and-stubby)
        + [Configuring and verifying stubby areas](#stubby)
    + [OSPFv3](#ospfv3)
        + [OSPFv3 traditional configuration](#v3-traditional)
        + [OSPFv3 address family configuration](#v3-addr-family)

## <span id="fundamentals"></span>Fundamental OSPF concepts

### <span id="basic-config"></span>Basic OSPF configuration

    !-- Router config mode

    router ospf <pid>
      router-id 0.0.0.1
      network <subnet> <wc mask> area <area ID>

    !-- Interface config mode

    int fa0/1
      ip ospf <pid> area <area ID>
      ip ospf priority <0-255>
    
<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="basic-show-cmds"></span>Basic show commands

    show ip ospf interface [brief]        !-- excl. passive interfaces
    show ip protocols                   !-- incl. passive interfaces
    show ip ospf neighbors [detail]
    show ip ospf database               !-- LSAs for connected areas

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="timers"></span>Hello and dead timers
By default, the hello interval is 10 seconds and the dead interval is 4 times Hello on LAN. Changing the Hello interval changes the dead interval, but you can set the dead interval directly.

    interface <if>
      ip ospf hello-interval <secs>
      ip ospf dead-interval <secs>
      ip mtu <value>                    !-- must match other end

The command below will set the dead interval to 1 second and the hello interval to 1/multiplier.

    interface <if>
      ip ospf dead-interval minimal hello-multiplier <num>

    !-- Verification

    show ip ospf interface <if>

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="wan-adjacencies"></span>OSPF neighbors and adjacencies on WANs
The network type determines many things. See OneNote. OSPF chooses the network type automatically based on the interface type, but you can change it manually. The types are `broadcast`, `non-broadcast`, `point-to-multipoint`, and `point-to-point`.

    interface <if>
      ip ospf network <type>

    !-- Verification of network type

    show ip ospf interface !-- FULL/ - for non-broadcast ntwks
    show ip ospf int <if>  !-- network type on 2nd line

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="virtual-links"></span>OSPF virtual links
Virtual links allow you to form neighborships between discontiguous routers in the same area. The area number in the `area virtual link` command is the transit area over which the packets flow between both routers. The transit area must not be stubby.

    router ospf <pid>
      area <num> virtual-link <remote RID>

Example

    !-- R1

    router ospf 1
      area 1 virtual-link 4.4.4.4
    interface loopback 1
      ip addr 1.1.1.1 255.255.255.0
      ip ospf 1 area 1

    !-- R2

    router ospf 4
      area 1 virtual-link 1.1.1.1
    interface loopback 1
      ip addr 4.4.4.4 255.255.255.0
      ip ospf 4 area 1

    !-- Verification

    show ip ospf virtual-links
    show ip ospf neighbor                !-- FULL if working
    show ip ospf neighbor detail <RID>

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="lsdb"></span> The OSPF link-state database (LSDB)

    show ip ospf database                   !-- summary of LSAs
    show ip ospf database router <rid>      !-- detailed type 1 LSA info
    show ip ospf database network <ntwk id> !-- detailed type 2 LSA info
    show ip ospf database summary <ntwk id> !-- detailed type 3 LSA info
    show ip ospf database asbr-summary      !-- detailed type 4 LSA info
    show ip route                           !-- O IA denotes interarea route
    show ip ospf | begin Area.1

### <span id="metric-tuning"></span> Metric tuning
Note the difference in units between reference bandwidth and the bandwidth command for interfaces

    auto-cost reference-bandwidth <bw in Mbps>  !-- config-router
    bandwidth <speed in Kbps>                   !-- config-if
    ip ospf cost <value>                        !-- config-if

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="advanced-concepts"></span> Advanced OSPF concepts

### <span id="route-filtering"></span> Route filtering
Filtering can only be done at ABRs and ASBRs b/c all routers in an area need to have the same LSDB.

#### <span id="filtering-type3"></span>Filtering type 3 summary LSAs

    area <num> filter-list prefix <name> {in | out} !-- config-router

    !-- example
    area 0 filter-list prefix dopes in   !-- don't let matching subnets into area 0
    area 2 filter-list prefix dopes out  !-- don't let matching subnets out of area 2

#### <span id="filtering-rt-table"></span>Filtering OSPF routes added to the routing table
Since you can't filter type 3 LSAs from only some routers in an area, you can filter the route so it doesn't get added to the routing table of some routers in an area. The command can refer to an ACL (named/numbered), prefix list, or route map.

    distribute-list prefix <name> in     !-- config-router

### <span id="summarization"></span> Route summarization
Summarization is done at ABRs and at least one subordinate route must be in its routing table for the summary route to be advertised.

    area <id> range <ip addr> <mask> [cost <cost>]

    !-- example

    area 0 range 10.16.0.0 255.255.255.252.0

#### <span id="asbr-summarization"></span>Manual summarization at ASBRs
The command below is for summary routers to create type 5 LSAs for redistributing.

    summary-address <prefix> <mask>

### <span id="default-and-stubby"></span> Default routes and stubby areas
The command below creates a type 5 LSA for the default route.

    default-information originate [always] [metric <value>] [metric-type <type>] [route-map <name>]

#### <span id="stuby"></span>Configuring and verifying stubby areas

    !-- stubby

    area <id> stub                          !-- all rtrs in area

    !-- totally stubby

    area <id> stub no-summary               !-- all ABRs
    area <id> stub                          !-- all other rtrs

    !-- set default route metric
    !-- can differ on ABRs

    area <id> default-cost <cost>           !-- ABRs only

    !-- NSSA

    area <id> nssa no-summary               !-- all ABRs
    area <id> NSSA                          !-- all other rtrs

    !-- Verification

    show ip ospf                            !-- "It is a stub area"
    show ip ospf database summary 0.0.0.0   !-- "summary Network Number"
    show ip ospf database database-summary  !-- Summary Net

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="ospfv3"></span> OSPFv3

### <span id="v3-traditional"></span>OSPFv3 traditional configuration

    ipv6 unicast-routing
    ipv6 cef                        !-- not req, but recommended
    ipv6 router ospf <pid>
    router-id <rid>                 !-- if no active IPv4 addr
    ipv6 ospf <pid> area <area>     !-- config-if
    
    !-- Verification

    show ipv6 ospf interface brief
    show ipv6 ospf neighbor
    show ipv6 ospf database

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="v3-addr-family"></span>OSPFv3 address family configuration

    router ospfv3 <pid>
    router-id <rid>
    address-family {ipv4|ipv6} unicast
      passive-interface <if>
      exit-address-family
    
    interface <if>
      ospfv3 <pid> {ipv4|ipv6} area <num>

In the example below, the passive interfaces were specified under config-router mode but were moved to the address family in the running-config.
    
    router ospfv3 1
    router-id 1.1.1.1
    address-family ipv4 unicast
      passive-interface Fa0/0
      passive-interface loop0
    exit-address-family
    address-family ipv6 unicast
      passive-interface Fa0/0
      passive-interface Loop0
      maximum-paths 32
    
    interface Fa0/1
      ip address 10.1.2.1 0.0.0.252
      ipv6 address 2002::1/64
      ospfv3 1 ipv6 area 0
      ospfv3 1 ipv4 area 0
    
    !-- Verification

    show ip route ospf
    show ipv6 route ospf
    show ospfv3 neighbor
    show ospfv3 interface brief
    show ofpsv3 database         !-- info for IPv4 and v6
    show ipv6...                 !-- traditional ipv6 commands

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>