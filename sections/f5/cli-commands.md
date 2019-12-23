---
title: "CLI commands"
layout: default
category: f5
order: 1
---

## High availability

    show cm sync-status
    list cm device-group [<name>]

    show cm traffic-group traffic-group-1 [all-properties]

    show sys ha-status all-properties

### Connection and persistence mirroring
    
    list ltm virtual <name> mirror
    show sys connection

    list ltm persistence <type> <name> mirror enabled

## Performance

    show sys performance all-stats historical
    
    for((i=1;i<=30;i+=1)); do echo "=================================================================="; clock; echo "=================================================================="; tmsh show sys performance all-stats historical; printf "\n\n"; sleep 30; done; printf "\nLoop done\n\n\n\n";

## Persistence

    show ltm persistence persist-records virtual <name> all-properties
    show ltm persistence persist-records client-addr <ip addr>
    delete ltm persistence persist-records virtual <name>

## Crypto

    list sys crypto cert <cert name>
    list sys crypto key <keyname>
    tmm --clientciphers '<list>'        # from bash

***

## CLI transactions

create cli transaction
modify ltm virtual some_vs profiles delete { tcp-profile-1}
modify ltm virtual some_vs profiles add { tcp-profile-2}
submit cli transaction

## Connecting to node to simulate HTTP monitor
You can run a monitor on a node that is not already monitored and see the results by issuing the command `run ltm monitor <type> <name> destination <ip addr:port>`. If the node is already monitored by the F5 using that monitor, it will complain. In that case, you can connect via Telnet and enter the send string manually to see what the answer it.

    telnet <ip addr> <port>
    GET /mystate.txt HTTP/1.1
    Host: example.com
    Connection: Close
    <press enter twice>

## tcpdump and tshark
[K411: Overview of packet tracing with the tcpdump utility](https://support.f5.com/csp/article/K411)

[K2289: Using advanced tcpdump filters](https://support.f5.com/csp/article/K2289)

[K13637: Capturing internal TMM information with tcpdump](https://support.f5.com/csp/article/K13637)

    tcpdump -nni 0.0:nnnp host <ip> and not tcp port 43 -w /var/tmp/file.pcap