---
title: "Advanced BGP concepts"
layout: default
category: routing
order: 30
---

## <span id="toc"></span> Table of contents
* [Internal BGP between Internet-connected routers](#ibgp)
    - [Examining BGP table entries for iBGP peers](#examine-ibgp-entries)
    - [Understanding next-hop reachability issues ](#ibgp-next-hop-issues)
* [Avoiding routing loops when forwarding toward the Internet](#routing-loops)
    - [Using an iBGP mesh](#ibgp-mesh)
    - [IGP redistribution and BGP synchronization](#bgp-sync)
* [BGP Route filtering](#route-filtering)
    - [BGP filtering overview](#bgp-filtering-overview)
    - [Inbound and outbound BGP filtering on prefix/length](#inbound-outbound-filtering)
* [Clearing BGP neighbors](#clearing-bgp-neighbors)
* [BGP Peer groups](#peer-groups)
* [BGP path attributes and best-path algorithm](#bgp-pa-and-best-path)
    - [BGP path attributes](#bgp-path-attribute)
    - [Overview of the BGP best-path algorithm](#bgp-best-path-algorithm)
* [Influencing an enterprise's outbound ](#influencing-outbound-routes)
    - [Influencing BGP weight](#bgp-weight)
    - [Setting the local preference](#bgp-local-pref)
    - [IP routes based on BGP best paths](#routes-based-on-best-path)
        + [BGP and the maximum-paths command](#bgp-max-paths)
    - [Increaasing the length of the AS_PATH ](#increase-as-path-length)
* [Influencing an enterprise's inbound routes with MED and AS Path prepending](#med-aspath)
    - [MED concepts](#med)
    - [AS path prepending](#aspath)

## <span id="ibgp"></span> Internal BGP between Internet-connected routers
When an enterprise has two or more Internet-connected routers using BGP with the ISP, they also need to exchange BGP routes with each other using iBGP so they know which has the best path for an Internet route.

Configuring iBGP is a matter of using the same ASN. If there's a single link between both routers, there's no need to use loopback addresses for the neighborship. If the routers are in different geographical locations, configure neighborships via loopback interfaces. iBGP uses a high TTL, so there's no need to configure multihop.

    router bgp 11
      no synchronization
      bgp log-neighbor-changes
      aggregate-address 128.107.0.0 255.255.224.0 summary-only
      redistribute ospf 1 route-map only-128-107
      neighbor 1.1.1.1 remote-as 11

Example

    !-- R1

    interface loopback 0
      ip addr 10.1.1.1 255.255.255.255
    router bgp 11
      neighbor 10.1.1.2 remote-as 11
      neighbor 10.1.1.2 update-source loopback0
    
    !-- R2

    interface loopback 1
      ip addr 10.1.1.2 255.255.255.255
    router bgp 11
      neighbor 10.1.1.1 remote-as 11
      neighbor 10.1.1.1 update-source loopback1

Verification

    show ip bgp summary
    show ip bgp neighbors <nghbr ip>

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="examine-ibgp-entries"></span> Examining BGP table entries for iBGP peers
To better understand the BGP table with two+ Internet-connected routers in the same company, start with one prefix and compare the BGP table entries of the two routers for that prefix.

    show ip bgp <ip addr>/<len>
    show ip bgp <ip addr>/<len> longer-prefixes

    !-- example

    show ip bgp 181.0.0.0/8 longer-prefixes

* Remember that iBGP peers don't add the ASN when advertising to each other.
* Routes learned internally will have an "i" as the third character.
* iBGP routers don't advertise iBGP-learned routes to iBGP peers.

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="ibgp-next-hop-issues"></span> Understanding next-hop reachability issues with iBGP
With IGPs, the IP routes in the routing table list a next-hop IP address, and this IP address tends to be in a common subnet. BGP-learned routes can have next hop IP addresses corresponding to a peer's loopback interface, since the advertising router lists its own update-source IP address as the next-hop address. Further, BGP peer B might indicate BGP peer A's IP address as the next hop when providing the route to BGP peer C. In that case, BGP peer C might not have a route to BGP A if they don't peer directly. The IP routing process can use routes whose next-hop addresses aren't in connected subnets as long as each router has an IP route matching the next-hop IP address. To ensure reachability to the next hop, do one of these two things:

* Create static routes or use an IGP so each router can reach these next-hop addresses that exist in other ASNs.
* Change the default iBGP behavior with the commannd<br> `neighbor <nghbr ip> next-hop-self`.

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="routing-loops"></span> Avoiding routing loops when forwarding toward the Internet
A typical enterprise network design uses default routes advertised by an IGP to steer traffic toward one or two Internet-connected routers. However, a routing loop can occur when Internet-connected routers don't have a direct connection to each other.

![BGP routing loop](#/img/bgp-loop.png)

This loop is caused by the lack of knowledge about the best route for the subnet for which the packet is destined. To avoid this, core1 and core2 need to know the best BGP routes. There are two solutions:

* Run BGP on at least some routers internal to the enterprise (e.g., core1 and core2) using an iBGP mesh
* Redistribute BGP routes into the IGP (not recommended)

### <span id="ibgp-mesh"></span> Using an ibgp mesh
Peering as shown below doesn't work because recall that __routers don't advertise routes learned from iBGP peers to other iBGP peers__. This behavior prevents routing loops.

![BGP routing loop](#/img/ibgp-bad-mesh.png)

You instead have to peer as shown below, so edge1 learns routes from edge2 that the core routers wouldn't share, and vice-versa.

In addition to the normal iBGP commands to set up the neighborships here using loopback interfaces, the edge routers would use the command `neighbor <ip> next-hop-self`

![BGP routing loop](#/img/ibgp-good-mesh.png)

__Config for edge1 and core1__
For the iBGP mesh to be complete, you'd have to add the edge2 and core2 configs.

    !-- edge1

    router bgp 11
      neighbor 10.100.1.2 remote-as 11
      neighbor 10.100.1.2 update-source loopback0
      neighbor 10.100.1.2 next-hop-self

      neighbor 10.100.1.3 remote-as 11
      neighbor 10.100.1.3 update-source loopback0
      neighbor 10.100.1.3 next-hop-self

      neighbor 10.100.1.4 remote-as 11
      neighbor 10.100.1.4 update-source loopback0
      neighbor 10.100.1.4 next-hop-self


    !-- core1

    router bgp 11
      neighbor 10.100.1.1 remote-as 11
      neighbor 10.100.1.1 update-source loopback0
      
      neighbor 10.100.1.2 remote-as 11
      neighbor 10.100.1.2 update-source loopback0

      neighbor 10.100.1.4 remote-as 11
      neighbor 10.100.1.4 update-source loopback0

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="bgp-sync"></span> IGP redistribution and BGP synchronization
Redistributing BGP into an IGP solves the issue of knowledge of the best exit points for Internet destinations but makes the IGP consume a lot more memory/processing and can crash the IGP if you're redistributing a large number of routes. BGP consumes less memory/CPU for a large number of routes compared to an IGP.

__Synchronization__ refers to the idea that iBGP-learned routes must be synchronized with IGP-learned routes for the same prefix before they can be used. That is, the sync feature tells the router not to consider an iBGP-learned route as the "best" unless the exact prefix was learned through an IGP and is currently in the routing table. IOS disabled synchronization in later IOS versions because most companies use iBGP meshes instead of redistribution. The setting can be enabled with `synchronization` and disabled with `no synchronization` config-router commands.

__Note:__ Redistributing from BGP to an IGP isn't recommended to exchange Internet routes. Redistributing from BGP to an IGP is common when using BGP for MPLS.

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="route-filtering"></span> Route filtering
BGP allows routers to filter inbound/outbound routes per neighbor. A BGP neighborship must be cleared for the filtering to take effect, done via the `clear` exec-mode command.

### <span id="bgp-filtering-overview"></span> BGP filtering overview
BGP filtering can match prefixes/length, and can also match many PAs. For example, BGP can check the AS_PATH PA for the number of ASNs or whether a particular ASN is/isn't on the list. Whereas with EIGRP the filtering applies to all outbound/inbound updates, with BGP filtering applies to neighborships.

#### BGP filtering tools

| BGP subcommand                     | What can be matched            |
| :---                               | :---                           |
| neighbor distribute-list <std ACL> | prefix + wc mask               |
| neighbor distribute-list <ext ACL> | prefix + len + wc mask         |
| neighbor prefix-list               | n bits of pfx + range          |
| neighbor filter-list               | AS_path contents               |
| neighbor route-map                 | pfx, len, AS_Path, + other PAs |

### <span id="inbound-outbound-filtering"></span>  Inbound and outbound BGP filtering on prefix/length
When an enterprise peers with with ISPs and exchanges BGP routes, one ISP might learn that the enterprise would be the best path to send traffic to another ISP. Engineers should filter routes sent to the ISP over the eBGP connection to avoid becoming a transit AS. Enterprises should also filter all private IP address ranges.

In the example below, the command `clear ip bgp <nghbr ip>` is a hard reset that brings the BGP neighborship down and back up so filtering can take effect.

    !-- check routes advertised before filtering

    show ip bgp neighbor <nghbr ip> advertised-routes
    
    !-- filter pvt addr

    conf t
    ip prefix-list only-public permit 128.107.0.0/19
    router bgp 11
      neighbor 1.1.1.1 prefix-list only-public out
      end
    !
    clear ip bgp 1.1.1.1
    
    !-- check routes advertised after filtering

    show ip bgp neighbor <nghbr ip> advertised-routes

__Alternatives to the example above__

    !-- using an ACL as a distribute-list

    access-list 101 permit ip host 128.107.0.0 host 255.255.224.0
    router bgp 11
      neighbor 1.1.1.1 distribute-list 101 out
    
    !-- same prefix list referenced by a route-map

    ip prefix-list only-public seq 5 permit 128.107.0.0/19
    route-map only-public-rmap permit 10
      match ip address prefix-list only-public
    router bgp 11
      neighbor 1.1.1.1 route-map only-public-rmap out

### Displaying the results of BGP filtering
In order to confirm the effects of filtering you need to see the before and after output.

    show ip bgp                                  !-- BGP table (no filters)
    show ip bgp neighbors <ip> advertised-routes !-- BGP updates sent post-filter

    show ip bgp neighbors <ip> received-routes   !-- rcvd b4 filters
    show ip bgp neighbors <ip> routes            !-- after inbound filters
  
The third command above requires the neighbor soft-reconfiguration inbound command.

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="clearing-bgp-neighbors"></span> Clearing BGP neighbors
As discussed, new BGP filters don't take effect until the neighbor relationship is cleared. There are different options, including the clear command already seen. A hard reset is defined as bringing down the neighborship, terminating the TCP connection, and removing all BGP table entries learned from that neighbor.  A soft reset does not bring down the neighborship or terminate the TCP connection, but the router resends outbound updates and processes incoming updates as per any current outbound/inbound filters.

### BGP clear command options

#### Hard resets:

    clear ip bgp *                   !-- all neighbors
    clear ip bgp <nghbr id>          !-- 1 neighbor

#### Soft resets:

    clear ip bgp <nghbr id> out      !-- 1 neighbor
    clear ip bgp <nghbr id> soft out !-- 1 neighbor
    
    clear ip bgp <nghbr id> in       !-- 1 neighbor
    clear ip bgp <nghbr id> soft in  !-- 1 neighbor
    
    clear ip bgp * soft              !-- all neighbors
    clear ip bgp <nghbr id> soft     !-- 1 neighbor
  
The `clear ip bgp <nghbr id> soft in` command is older and only works if the `neighbor <nghbr id> soft-reconfiguration inbound` command was configured for that neighbor. This requires more memory. The newer `clear ip bgp <nghbr id> in` command doesn't require that other command and instead uses the BGP route refresh feature, which asks a neighbor to resend its full BGP update so the local router can apply its current inbound BGP filters and update the BGP table.

The last pair of command does a soft reset on both inbound and outbound at once.

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="peer-groups"></span> BGP Peer groups
By default, IOS creates BGP updates on a neighbor-by-neighbor basis. More neighbors means more CPU resources used. Non-default settings applied to each neighbor further increases CPU consumption. IOS allows you to create BGP peer groups of neighbors with similar characteristics so you can apply your non-default configurations to the groups and thereby decrease the CPU requirements. Note that since BGP forms individual TCP sessions with neighbors, individual updates are still sent.

Example. The point of the example below is to keep the router from learning private IP address ranges.

    router bgp 1234
      bgp log-neighbor-changes
      network 192.0.2.0
      network 198.51.100.0 mask 255.255.255.252
      network 198.51.100.4 mask 255.255.255.252
      neighbor ROUTE-PG peer-group
      neighbor ROUTE-PG prefix-list ROUTE-DEMO in
      neighbor 198.51.100.2 remote-as 2345
      neighbor 198.51.100.2 peer-group ROUTE-PG
      neighbor 198.51.100.6 remote-as 3456
      neighbor 198.51.100.6 peer-group ROUTE-PG
    
    ip prefix-list ROUTE-DEMO seq 5 deny 10.0.0.0/8 le 32
    ip prefix-list ROUTE-DEMO seq 10 deny 172.16.0.0/12 le 32
    ip prefix-list ROUTE-DEMO seq 15 deny 192.168.0.0/16 le 32
    ip prefix-list ROUTE-DEMO seq 20 permit 0.0.0.0/0
    ip prefix-list ROUTE-DEMO seq 25 permit 0.0.0.0/0 ge 8

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="bgp-pa-and-best-path"></span> BGP path attributes and best-path algorithm
BGP has many path attributes that are used solely for the best-path algorithm and some that are used for other purposes. The term BGP best-path algorithm refers to the process by which BGP chooses the best route on a router by examining the competing paths in its BGP table.

### <span id="bgp-path-attribute"></span> BGP path attributes
BGP PAs define facts about a particular route through a network. Here are a few.

* __Autonomous System Path (AS_Path):__ Lists ASNs in the end-to-end path. Used mainly for loop prevention. E.g., if a router receives a route with its own ASN in the path it will ignore it.
* __Next-hop IP address (Next_Hop):__ Used in the best-path algorithm.
* __Weight:__ Not a PA, but num value used by Cisco for its best-path implementation. Not advertised to peers.
* __Local Preference (Local_Pref):__ Num value 0-232-1, set and sent through single AS to influence choice of best route for all routers inside the AS.
* __AS_Path (length):__ Number of ASNs in AS_Path PA.
* __Origin:__ Value indicating the route was injected into BGP (I=IGP; E=EGP; ?=incomplete info)
* __Multi-Exit Discriminator (MED):__ Set and advertised within an AS, impacting the BGP decision of routers in the other AS. Smaller is better.

#### Finding BGP PAs in the show ip bgp command output

    router#show ip bgp
    BGP table version is 4108, local router ID is 199.220.5.234
    Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                  r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
                  x best-external, a additional-path, c RIB-compressed,
    Origin codes: i - IGP, e - EGP, ? - incomplete
    RPKI validation codes: V valid, I invalid, N Not found

         Network          Next Hop            Metric LocPrf Weight Path
     *>  8.50.0.64/28     10.0.15.30                             0 3549 53345 i
     *>i 8.100.0.0/24     10.15.30.45              0    100      0 25674 i
     *>  10.150.0.0/16    10.30.45.60                   500      0 3549 3549 i

### <span id="bgp-best-path-algorithm"></span> Overview of the BGP best-path algorithm
The steps below aren't numbered in the RFC, but are numbered here for ease of study. Don't worry about memorizing the sequence shown here.

0. Next hop reachable?
1. __Weight:__ Prefer bigger
2. __Local_Pref:__ Prefer bigger
3. __Locally injected routes:__ Better than iBGP/eBGP learned
4. __AS_Path length:__ Prefer smaller
5. __origin:__ Prefer I over E
6. __MED:__ Prefer smaller
7. __Neighbor type:__ Prefer eBGP over iBGP
8. __IGP metric to Next_Hop:__ Prefer smaller 

BGP goes through these steps in order until it finds the best path. If multiple paths tie, BGP performs other checks because unlike IGPs, it needs a single best path. The other checks are oldest eBGP route, lowest neighbor BGP RID, and lowest neighbor address.

BGP first takes the two oldest routes and compares them. It then takes the winner and compares it with the next oldest route until it's considered all of them.

The CCNP ROUTE exams focuses on the following four paths because they allow engineers to influence the choice of best path:

* Weight
* Local_Pref
* AS_Path Length
* MED (often called metric)

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="influencing-outbound-routes"></span> Influencing an enterprise's outbound routes

### <span id="bgp-weight"></span> Influencing BGP weight
A Cisco router can use the BGP weight on that router to influence its choice of outbound route. The weight can be set selectively per incoming route using a route map or for all routes from a neighbor. The range is 0 - 65,535. The default is 0 for learned routes and 32,768 for locally-injected routes. You cannot define a new default.

    neighbor <nghbr ip> route-map <name>     !-- per prefix
    neighbor <nghbr ip> weight <weight>      !-- per neighbor

Examples

    !-- using a route map

    ip prefix-list match-101 permit 101.0.0.0/8
    
    route-map set-weight-50 permit 10
      match ip address prefix-list match-101
      set weight 50
    route-map set-weight-50 permit 20
    
    router bgp 11
      neighbor 192.168.1.2 route-map set-weight-50 in
      end
    clear ip bgp 192.168.1.2 soft
    
    !-- using the neighbor weight command

    router bgp 11
      neighbor 1.1.1.1 weight 60
      end
    clear ip bgp 1.1.1.1 soft
    show ip bgp <ntwk id>/<pfx> longer-prefixes

### <span id="bgp-local-pref"></span> Setting the local preference
Local_Pref is a PA that identifies the best exit point from the AS to reach a given prefix. It ranges from 0 - 4294967295 (232-1). Local_Pref can be set by routers as they receive eBGP updates. The Local_Pref can then be shared with iBGP neighbors so the entire enterprise agrees on the best path toward that route. The default is 100. You can change the default with the bgp default local-preference <num> BGP subcommand. You can change configure the local preference via a neighbor route map with direction of in. Local_Pref has its own column in the BGP table.

Example

    ip prefix-list match-184 seq 5 permit 184.0.0.0
    ip prefix-list match-185 seq 5 permit 185.0.0.0
    
    route-map set-LP-150 permit 10
      match ip address prefix-list match-185
      set local-preference 150
    route-map set-LP-150 permit 15
      match ip address prefix-list match-184
      set local-preference 50
    
    router bgp 11
      neighbor 1.1.1.1 route-map set-LP-150 in
      end
    !
    clear ip bgp 1.1.1.1 soft
    
    !-- verification

    show ip bgp <nghbr subnet>/<len> longer-prefixes !-- shows weight
    show ip bgp <subnet id>/<len>                    !-- omits weight

### <span id="routes-based-on-best-path"></span> IP routes based on BGP best paths
BGP doesn't add routes to the IP routing table directly, but rather gives them to the IOS Routing Table Manager (RTM) for consideration. The RTM chooses the best route among many competing sources (connected, BGP, IGP). The RTM uses the concept of administrative distance to choose the best route among competing sources.

In the event that the same route is learned via BGP and some other way, such as when doing redistribution with MPLS VPN, the command `show ip bgp rib-failures` can be helpful. It lists routes for which BGP has chosen a best route that was not added to the routing table (also called the Routinig Information Base).

#### <span id="bgp-max-paths"></span> BGP and the maximum-paths command
BGP supports the `maximum-paths` command, but it works much differently: If after going the entire best-path algorithm multiple routes tie, BGP will add them to the routing table up to and incl. the number indicated by the maximum-paths command.

### <span id="increase-as-path-length"></span> Increaasing the length of the AS_PATH using AS_Path prepend
You can add ASNs to the AS_Path to make a route less likely to be chosen as best without hampering the PA's loop-prevention role. For loop prevention purses, use an ASN already on the list or the organization's ASN.

    !-- the following adds ASN 3 twice

    route-map add-two-asns permit 10
      set as-path prepend 3 3
    router bgp 11
      neighbor 192.168.1.6 route-map add-two-asns in

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

***

## <span id="med-aspath"></span> Influencing an enterprise's inbound routes with MED and AS path prepending
An enterprise has very little control over inbound routes, as it doesn't control the PE equipment. However, there's a tool that provides some control over the last ASN hop between an ISP and enterprise customer. This tool is  called __Multi-Exit Discriminator (MED)__ and works with dual-homed and dual-multihomed designs.

### <span id="med"></span> MED concepts
WIth a dual-homed design there are two links between the enterprise and the ISP. The enterprise can advertise a MED value to the ISP to indicate which route is best. Smaller number = better.

MED is a PA that allows an AS to tell a neighboring AS the best way to forward packets into it. The MED is propagated inside the AS it was sent to but is not advertised further. It ranges from 0 - 4,294,967,295 (2^32-1). Smaller is better, and the default is 0.

    route-map set-med-to-I1-1 permit 10
      match ip address prefix-list only-public
      set metric 10
    
    route map set-med-to-I1-4 permit 10
      match ip address prefix-list only-public
      set metric 20
    
    ip prefix-list only-public permit 128.107.0.0/19
    
    router bgp 11
      neighbor 1.1.1.1 route-map set-med-toI1-1 out
      neighbor 192.168.1.2 route-map set-med-toI1-4 out
    
    !-- verification

    show ip bgp

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>

### <span id="aspath"></span> AS path prepending
By prepending your AS number to the AS_PATH of your outgoing eBGP updates to the undesired upstream ISP, the path appears longer and therefore less desirable.

Configuring AS path prepending requires a route map with the `set as-path prepend` command. The route map will be applied to outbound updates to the BGP peer. AS path prepending doesn't work with iBGP.

Example

    router bgp 64496
      no synchronization
      bgp log-neighbor-changes
      network 10.1.1.0 mask 255.255.255.0
      neighbor 10.1.2.2 remote-as 64511
      neighbor 10.1.2.2 route-map AS64511_out out
    !
    route-map AS64511_out permit 10
      set as-path prepend 64496 64496

<div style="text-align: right;"><a href="#toc">Back to ToC</a></div>
