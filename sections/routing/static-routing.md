---
title: "Static routing"
layout: default
category: routing
order: 5
---

Static routes have an AD of 1, which makes them preferred to all routes except directly connected routes. You can specify an AD so it will not be preferred over routes from routing protocols.

    ip route 210.10.5.0 255.255.255.0 1.1.1.1 250

You can also make static routes conditional by using IP SLA and tracking objects.

    ip sla 10
      icmp-echo 192.168.70.1
      frequency 5
      exit
    ip sla schedule 10 life forever start-time now
    !
    track 10 ip sla 10 reachability
    !
    ip route 210.10.5.0 255.255.255.0 192.168.70.1 250 track 10

Another thing about static routes is that they can be recursive. You can, for example, have a route from router A to router D that says the next hop is router C. You can have a route to router C that says the next hop is B, with B being directly connected. When you have a packet destined for router D, router A will keep looking up routes until it knows to send the packet to router B.

    +---+    +---+    +---+    +---+
    | A |----| B |----| C |----| D |
    +---+    +---+    +---+    +---+