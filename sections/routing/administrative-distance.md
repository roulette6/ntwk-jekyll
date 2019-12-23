---
title: "Administrative distance"
layout: default
category: routing
order: 1
---

When IOS must choose among routes learned through different routing protocols, it uses administrative distance (AD) to decide which route to add to the routing table. The AD denotes how believable a routing protocol is to a router. Changes to the default AD are local and cannot be communicated to other routers.

You can add a high AD to a static route so it's only used if a dynamic routing protocol doesn't find a route.

    ip route 192.168.0.0 255.255.255.0 10.1.130.1 210

## Default admistrative distances

| Route type              | AD   |
| :---                    | ---: |
| Connected               | 0    |
| Static                  | 1    |
| BGP (external routes)   | 20   |
| EIGRP (internal routes) | 90   |
| IGRP                    | 100  |
| OSPF                    | 110  |
| IS-IS                   | 115  |
| RIP                     | 120  |
| EIGRP (external routes) | 170  |
| BGP (internal routes)   | 200  |
| Unusable                | 255  |
