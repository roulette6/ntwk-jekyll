---
title: "BGP introduction"
layout: default
category: routing
order: 25
---

## BGP introduction
* __Border Gateway Protocol__ is the routing protocol of the global Internet.
* BGP has a robust best-path algorithm that goes beyond just a lower metric.
* BGP neighbors don't need to be in the same subnet
* BGP uses TCP port 179 to pass BGP messages, which could pass through other routers.
* BGP uses path attributes (PAs), which are exchanged via router updates.

### BGP ASNs and the AS_SEQ path attribute
BGP uses path attributes to choose the best path and for other purposes. If no path attributes have been set, BGP uses the __AS_PATH__ attribute when choosing among competing routes. The AS_PATH PA comprises several components, one of which is the AS_SEQ.

The integer BGP ASN uniquely identifies one autonomous organization. Every company whose enterprise network connects to the Internet can get a BGP ASN, handed out by IANA/ICANN.

When a router advertises a route with BGP, the prefix/length is associated with a list of PAs, including the AS_PATH. The AS_PATH PA lists the ASNs that would be part of the end-to-end route for that prefix.

#### BGP uses the AS_PATH to perform two key functions:

* Choose the best route for a prefix based on the shortest AS_PATH
* Prevent routing loops by ignoring routes that list its own ASN in the AS_PATH.

#### Advertisement of prefix updates to other ASNs

* Enterprise router advertises its network and adds its own ASN to AS_PATH so it becomes an array of ASNs.

### Internal and external BGP
Internal vs. external BGP refers to whether a neighbor is in the same ASN. BGP routers behave differently based on whether they're iBGP or eBGP peers. For example, iBGP peers do not add their own ASN to the AS_PATH PA when advertising internally.

### Public and private ASNs
ASNs have to be unique so BGP loop prevention won't prevent parts of the Internet from being advertised. IANA controls and assigns ASNS. The ASN is 16 bits, which means the highest number is 65,535.

#### ASN assignment categories from IANA

| ASN             | Category                          |
| :---            | :---                              |
| 0               | Reserved                          |
| 1 - 64,495      | Assigned for public use           |
| 64,496 - 64,511 | Reserved for use in documentation |
| 64,512 - 65,534 | Private use                       |
| 65,535          | Reserved                          |


## BGP neighbor states
* __Idle:__
    - Trying to initiate a TCP connection and listening for TCP connections from peers
    - Admin disabled (neighbor shutdown)
    - Router is waiting before next retry
* __Connect:__
    - BGP initiates the TCP 3-way handshake.
    - If successful, send Open msg and change to OpenSent.
    - If failed, reset the ConnectRetryTimer and move to __active__ state.
* __Active:__
    - Trying to establish TCP connection
    - No BGP messages yet sent
* __OpenSent:__
    - TCP connection exists
    - This router has sent the first message to establish the BGP neighbor relationship
* __OpenConfirm:__
    - TCP connection exists
    - Other router has received an Open message but may still reject the relationship
* __Established:__
    - After receiving a a keepalive message from peer, the routers are neighbors and can exchange update messages

***

## Configuring BGP

### add a discard route to share a classful network
Do this if you want to advertise a larger prefix than what is in the routing table. ISPs prefer to receive larger prefixes to have an overall smaller routing table.

    ip route <classful network> <mask> null0

### Basic BGP configuration

    router bgp <asn>
      bgp router-id <rid>
      neighbor <ip addr> remote-as <asn>  !-- define a BGP neighbor and its ASN
      neighbor <ip addr> shutdown         !-- suspend neighborship w/o deleting it
      no neighbor <ip addr> shutdown      !-- resume neighborship w/o deleting it
      network <ip addr> [mask <mask>]     !-- route to share. Must be in routing table

### Configuring loopback interfaces with BGP

1. Configure loopback interface on each router
2. Tell BGP on each router to use the loopback IP address as the source
3. Configure the BGP neighbor command on each router to refer to the other router's loopback IP address
4. Make sure each router has routes to the neighbor's loopback interface
5. Configure eBGP multihop if several hops away, or disable whether a neighbor one hops away is known via a connected route.

__Configuration__
    
    router bgp
      neighbor <neighbor-ip> update-source <loopback if id>
      neighbor 100.1.1.2 update-source loopback 1
      neighbor <neighbor's loopback ip addr> remote-as <asn>
      neighbor <neighbor ip addr> ebgp-multihop <hops>
      !
      !-- or
      !
      neighbor <nghbor ip> disable-connected-check


## Verification and troubleshooting commands

    show tcp brief                 !-- TCP connex at this router incl. BGP
    show tcp summary               !-- a line of info for each TCP connex
    show ip bgp                    !-- BGP table
    show ip bgp summary            !-- basic config for local RT; 1 ln per BGP peer
    show ip bgp neighbors <addr>   !-- detailed info about neighbor state
    show ip bgp neighbors <addr> received-routes