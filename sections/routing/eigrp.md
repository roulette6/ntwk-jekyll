---
title: "EIGRP"
layout: page
category: routing
order: 15
---

## <span id="toc"></span> TOC
* [Basic EIGRP](#basic)
    - [Configuring the EIGRP router ID and passive interfaces](#rid)
    - [Configuring Hello and Hold timers](#timers)
    - [Configuring static neighborships](#stat-neighbor)
    - [Stats of hellos/updt/qry/rply/ACK sent/rcvd](#stats)
    - [EIGRP authentication](#auth)
* [Advanced EIGRP concepts](#advanced)
    - [Split horizon](#split-hor)
    - [EIGRP WAN bandwidth control](#wan-bw)
    - [EIGRP metric tuning](#metric-tuning)
        + [Configuring bandwidth and delay](#bw-dly)
        + [Configuring metric K-values](#k-values)
        + [Configuring offset lists](#offset)
    - [Optimizing EIGRP convergence](#optimize-convergence)
        + [Limit EIGRP Query messages by setting stub routers](#stub)
        + [Change the active timer](#active-timer)
    - [Route filtering](#route-filtering)
    - [Route summarization](#summarization)
    - [Default routes](#default-routes)
    - [Variance&mdash;Unequal metric route load sharing](#variance)
* [EIGRP for IPv6 and named EIGRP](#ipv6-named)
    - [Configuring EIGRP for IPv6](#ipv6)
    - [Configuring named EIGRP](#named-eigrp)

## <span id="basic"></span>Basic EIGRP
* EIGRP uses Reliable Transfer Protocol to ensure neighbors receive updates.
* EIGRP uses multicast IP address 224.0.0.10 and IP Protocol 88.
* Metric = ((10^7 &divide; least bw) + cumulative delay) &times; 256, where cumulative delay is measured in tens of &mu;seconds.
* EIGRP follows three steps to add routes to the routing table:
  1. __Neighbor discovery:__ Send Hello messages to discover potential neighboring EIGRP routers. Become neighbors after performing basic parameter checks. Must be in same subnet. 
  2. __Topology exchange:__ Neighbors exchange full topology updates when the neighbor relationship comes up and only partial updates based on network topology changes.
  3. __Choosing routes:__ Each router analyzes its respective EIGRP topology table, choosing the lowest-metric route to reach each subnet.


<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="rid"></span>Configuring the EIGRP router ID and passive interfaces
EIGRP doesn't require router IDs or for them to be unique, but it's good to have them and for them to be unique when injecting external routes into EIGRP.

If no wc mask is provided in a `network <ntwk id>` command, the network is treated as classful and any matching interfaces will be added.

    router eigrp <asn>
      eigrp router-id <id>
      [no] passive-interface <if>
      passive-interface default

    !-- Verification

    show ip protocols

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="timers"></span>Configuring Hello and Hold timers
The `ip hold-time` command is an instruction to neighbors on that interface, not the holdtime used by the local router. The default hold timer is 15 seconds on LAN and 180 seconds on T1 or lower. Hello and hold times do not need to match on neighbors.

Note that IOS doesn't prevent you from configuring a hello interval greater than the hold time, which will cause the neighborship to continuously flap.

    int fa0/1
      ip hello-interval eigrp <asn> <secs>
      ip hold-time eigrp <asn> <secs>

    !-- verification

    show run interface <if>
    show ip eigrp interface detail <if>   !-- old IOS vers might not show hold time
    show ip eigrp neighbors               !-- look at Hold Uptime

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="stat-neighbor"></span>Configuring static neighborships
Configuring static neighborships is useful for reducing multicast messages or in NBMA networks.

    router eigrp <asn>
      network <ntwk id> [<wc mask]
      neighbor <neighbor ip addr> <outgoing if>

    !-- verification
    !-- "static neighbor" under IP addr

    show ip eigrp neighbors detail

#### Caveats to static neighborships
* Neighbors must be in the same subnet as the interface indicated or the command will be rejected.
* When an interface is configured for static neighborships, EIGRP stops processing multicast packets on that interface: Dynamic neighbors will not be discovered nor continue to work.

### <span id="stats"></span>Stats of hellos/updt/qry/rply/ACK sent/rcvd

    show ip eigrp traffic

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="auth"></span>EIGRP authentication
EIGRP supports MD5 authentication. EIGRP for IPv6 is set up the same; just sub "ip" with "ipv6."

    key chain <name>
      key <num>
        key-string <string>
        [accept-lifetime ...]
        [send-lifetime ...]
        exit
      exit
    !
    interface <if>
      ip authentication mode eigrp <asn> md5
      ip authentication key-chain eigrp <asn> <chain name>
      end

    show key chain
    show ip eigrp neighbors

***

## <span id="advanced"></span>Advanced EIGRP concepts
* A router's EIGRP process seeds the topology table using these three sources:
    1. Prefixes of connected subnets for interfaces on which EIGRP has been enabled.
    2. Prefixes of connected subnets for interfaces referenced in an EIGRP neighbor command
    3. Prefixes learned by redistribution of routes into EIGRP from other routing protocols or routing information sources.
* EIGRP uses the following five protocol messages
    - Hello
    - Update
    - Query
    - Reply
    - Ack
* EIGRP uses Update and Ack messages as part of the topology exchange process. The update messages contain the following information:
    - Prefix
    - Prefix length
    - Metric components (bw, delay, reliability, and load)
    - Nonmetric items: MTU and hop count
* You can see the bandwidth and cummulative delay for a prefix with the command `show ip eigrp topology <ntwk id>/<pfx len>`.
* EIGRP neighbors only send full updates upon establishing a neighborship, including when a neighborship fails and recovers.
* When calculating metrics, the __Feasible Distance__ is the metric from the local router's perspective, whereas the __Reported Distance__ is the metric from the neighboring router's perspective.

### <span id="split-hor"></span>Split horizon
Omitting `eigrp <asn>` applies the command to RIP instead of to EIGRP.

    int fa0/1
      no ip split-horizon eigrp <asn>

    !-- verification
    show run interface <if>
    show ip eigrp interfaces detail <if>  !-- split horizon is enabled/disabled

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="wan-bw"></span>EIGRP WAN bandwidth control

    int fa0/1
      ip bandwidth-percent eigrp <asn> <percent>

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="metric-tuning"></span>EIGRP metric tuning

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

#### <span id="bw-dly"></span>Configuring bandwidth and delay
delay is displayed in microseconds in `show int`

    int fa0/1
      bandwidth <kbps>
      delay <tens of microseconds>

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

#### <span id="k-values"></span>Configuring metric K-values
Cisco strongly recommends you leave the k values alone, but you can set them in the lab to simplify the metric. The first value in the `metric weights` command is a QoS setting and **cannot** be changed. The other values are k1 through k5.

    router eigrp 1
      metric weights <ToS> K1 K2 K3 K4 K5  !-- K1 and K5 default to 1

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

#### <span id="offset"></span>Configuring offset lists`

    router eigrp 1
      offset-list <ACL number/name> {in|out} <offset> [<interface>]

    !-- Example (use the ntwk ID)

    access-list 11 permit 10.11.1.0
    !
    router eigrp 1
      offset-list 11 in 3 Serial0/0/0.1

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="optimize-convergence"></span>Optimizing EIGRP convergence
The best route is the successor and goes on the routing table. Loop-free backup routes that are usable are kept in the topology table as FS __if their RD is less than the successor's FD__

The command below will show you all routes, even non-FS

    show ip eigrp topology all-links

#### <span id="stub"></span> Limit EIGRP Query messages by setting stub routers
Stub routers do not forward traffic between two remote EIGRP-learned subnets. Stub routers do not advertise EIGRP-learned routes from one neighbor to other EIGRP neighbors. More significantly, Non-stub routers note which neighbors are stub routers and do not send them EIGRP queries.

    !-- config-router mode

    eigrp stub                           !-- advert connected and summary
    eigrp stub connected                 !-- advert conn routes
    eigrp stub summary                   !-- advert summary routes
    eigrp stub static                    !-- advert static routes (if..?)
    eigrp stub leak-map <name>           !-- advert routes specified by a leak-map
    eigrp stub redistributed             !-- advert redist routes if redist conf
    eigrp stub receive-only              !-- advert no routes

    !-- verification

    show ip eigrp neighbors detail       !-- "Stub peer advertising..."

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

#### <span id="active-timer"></span>Change the active timer
This is the timer the router uses to wait for EIGRP replies to its queries.

    router eigrp 1
      timers active-time <mins>

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="route-filtering"></span>Route filtering

Filtering with ACLs

    access-list 2 deny 10.17.32.0 0.0.31.255
    access-list 2 permit any
    !
    router eigrp 1
      distribute-list 2 out

    !-- verification

    show ip route
    show ip protocols                    !-- Outgoing update filter list

Filtering with IP prefix lists

    !-- the ge and le params are optional. See notes and practice.
    !-- The logic will likely be tested

    ip prefix-list <name> [seq <seq num>] {deny | permit} <prfx/len> [ge <value>] [le <value>]

    !-- example

    ip prefix-list fred seq 5 deny 10.17.35.0/24 ge 25 le 25
    ip prefix-list fred seq 10 deny 10.17.36.0/24 ge 26 le 26
    ip prefix-list fred seq 15 deny 0.0.0.0/0 ge 30 le 30
    ip prefix-list fred seq 20 permit 0.0.0.0/0 le 32

    router eigrp 1
    network 10.0.0.0
    distribute-list prefix fred out

Filtering by using route maps

    !-- create a prefix list to be referenced by the route map
    ip prefix-list manufacturing seq 5 permit 10.17.35.0/24 ge 25 le 25
    ip prefix-list manufacturing seq 10 permit 10.17.36.0/24 ge 26 le 26

    !-- create a route map
    route-map filter-man, deny 10
      match ip address prefix-list manufacturing
    route-map filter-man permit 20

    !-- apply to EIGRP
    router eigrp 1
    distribute-list route-map filter-man in

    !-- verification
    show ip protocols | b eigrp
    show route-map               !-- shows commas that weren't input

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="summarization"></span>Route summarization

    int fa0/1
      ip summary-address eigrp <asn> <prefix> <subnet-mask>
      ip summary-address eigrp 1 10.16.0.0 255.255.0.0

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="default-routes"></span> Default routes

Advertising the default route

    ip route 0.0.0.0 0.0.0.0 <if>    !-- create default route
    router eigrp 1
      network 0.0.0.0                  !-- inject into EIGRP topology database

Configuring a default network

    !-- configure the default ntwk and advert via EIGRP

    config t
    ip default-network <ntwk number>

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="variance"></span> Variance&mdash;Unequal metric route load sharing
Cisco IOS allows multiple routes for the same subnet and load balances based on dest IP. This has the added benefit of faster convergence because a route failure doesn't require a search for a new successor. To help add more routes than are unlikely to have the same FD, you can configure the variance multiplier, which is multiplied by the successor's FD. Any other FD =< this product is considered equal.

__Routes that are not feasible successors because their RD is > the successor's FD can never be added to the routing table regardless of variance settings.__

    router eigrp 666
      maximum-paths <num>       !-- default is 4
      variance <multiplier>     !-- 1-128

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="ipv6-named"></span>EIGRP for IPv6 and named EIGRP

### <span id="ipv6"></span>Configuring EIGRP for IPv6
Remember that for EIGRP for IPv6 to derive a router ID automatically, there must be a loopback or physical interface in an up/up state with an IPv4 address assigned.

    ipv6 unicast-routing
    ipv6 router eigrp <asn>
      router-id <rid>                        !-- if there's no IPv4 addr configured
      no shut                                !-- if EIGRP is shut

    interface <if>
      ipv6 eigrp <asn>                       !-- IPv6 must be enabled

    !-- Verification

    show ipv6 route
    show ipv6 route eigrp
    show ipv6 route <pfx/len>
    show ipv6 protocols                      !-- ifs enabled, k vals, redstrb,
                                             !-- max-paths, admin dst
    show ipv6 eigrp neighbors
    show ipv6 eigrp <asn> interface detail   !-- hello/hold intervals
    show ipv6 eigrp topology [all-links]
    show ipv6 eigrp traffic                  !-- stats on EIGRP msgs sent/rcvd
    debug ipv6 eigrp notifications           !-- sent/rcvd updates

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="named-eigrp"></span>Configuring named EIGRP

    router eigrp <v instance name>
      address-family ipv4 autonomous-system <asn>
        network <ntwk id> [<wc mask>]
        eigrp stub
        eigrp router-id <id>
        af-interface {default|<if>}
          hello-interval <secs>
          hold-time <secs>
        exit
        topology base
          maximum-paths <num>
          variance <multiplier>

Sample traditional EIGRP config

    interface FastEthernet0/0
      ip address 172.16.1.1 255.255.255.0
      ipv6 address 2001::1/64
      ipv6 eigrp 2

    interface Serial1/0
      ip address 10.1.1.1 255.255.255.252
      ip hello-interval eigrp 1 2
      ip hold-time eigrp 1 10
      ipv6 address 2002::1/64
      ipv6 eigrp 2

    router eigrp 1
      variance 2
      network 0.0.0.0
      passive-interface default
      no passive-interface Serial1/0

    ipv6 router eigrp 2
      variance 2

Equivalent EIGRP named config

    router eigrp R1DEMO
      address-family ipv4 unicast autonomous-system 1
        af-interface default
          hello-interval 2
          hold-time 10
          passive-interface
        exit-af-interface

        af-interface Serial1/0
          no passive-interface
        exit-af-interface

        topology base
          variance 2
        exit-af-topology

        network 0.0.0.0
      exit-address-family

      address-family ipv6 unicast autonomous-system 2
        topology base
          variance 2
        exit-af-topology
      exit-address-family

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>