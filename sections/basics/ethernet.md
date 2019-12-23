---
title: "Ethernet"
layout: page
category: basics
order: 2
---

## Variety of physical Ethernet standards

| Speed     | Common Name      | Informal IEEE Standard Name | Formal IEEE Standard Name | Cable Type, Max Length |
|:---       |:---              |:---                         |:---                       |:---                    |
| 10 Mbps   | Ethernet         | 10BASE-T                    | 802.3                     | Copper, 100 m          |
| 100 Mbps  | Fast Ethernet    | 100BASE-T                   | 802.3u                    | Copper, 100 m          |
| 1000 Mbps | Gigabit Ethernet | 1000BASE-LX                 | 802.3z                    | Fiber, 5000 m          |
| 1000 Mbps | Gigabit Ethernet | 1000BASE-T                  | 802.3ab                   | Copper, 100 m          |
| 10 Gbps   | 10 Gig Ethernet  | 10GBASE-T                   | 802.3an                   | Copper, 100 m          |

## Ethernet frame format
1. __Preamble:__ (header) 7 bytes; synchronization
2. __Start Frame Delimiter (SDF):__ (header) 1 byte; indicates that next byte marks destination MAC address field
3. __Destination MAC address:__ (header) 6 bytes; intended recipient
4. __Source MAC address:__ (header) 6 bytes; sender
5. __Type:__ (header) 2 bytes; Defines the protocol (usually IPv4 or IPv6)
6. __Data and pad:__ 46-1500 bytes; data from a higher layer, plus padding if necessary for minimum length of 46 bytes
7. __Frame Check Sequence (FCS):__ (trailer) 4 bytes; allows receiving NIC to chek frame for transmission errors

## Half-duplex
Half-duplex devices can only send or receive data. Sending and receiving simultaneously results in a __collision__. Half-duples devices listen for collisions and send a jamming signal when they occur, to let other devices know. Devices then wait a random amount of time and try sending again. This is the __carrier sense multiple access with collision detection (CSMA/CD)__ algorithm.