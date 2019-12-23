---
title: "Infrastructure Services"
layout: default
category: infrastructure
order: 10
---

* [Syslog](#syslog)
* [Network Time Protocol (NTP)](#ntp)
* [CDP and LLDP](#cdp)

## Syslog<span id="syslog"></span>
Syslog is defined in [IETF RFC 5424](https://tools.ietf.org/html/rfc5424). Log messages have a severity level to indicate their importance.

|Keyword | Numeral | Description | Category |
| :---   | :---    | :---        | :---     |
| Emergency     | 0 | System unstable               | Severe |
| Alert         | 1 | Immediate action required     | Severe |
| Critical      | 2 | Critical event (highest of 3) | Impactful |
| Error         | 3 | Error event (middle of 3)     | Impactful |
| Warning       | 4 | Warning event (lowest of 3)   | Impactful |
| Notification  | 5 | Normal, more important        | Normal |
| Informational | 6 | Normal, less important        | Normal |
| Debug         | 7 | Requested by user debug       | Debug |

### Log message format

    *Dec 18 17:10:15.079: %LINEPROTO-5-UPDOWN: Line protocol on Interface 
    FastEthernet0/0, changed state to down

The above log message has the following components:

* A timestamp
* The router facility that generated the message: %LINEPROTO
* The severity level: 5
* A mnemonic for the message: UPDOWN
* The description: Line protocol on Interface FastEthernet0/0, changed state to down

### Configuring syslog

Logging to terminal when connected via console

    config t
    logging console [<sev lvl/name>]
    no logging console

VTY users don't see log messages by default. The command below allows VTY users to see syslog mesages for that session.

    terminal monitor

The command below allows you to always log to VTY lines.

    config t
    logging monitor

The command below sets up logging messages to the buffer for later review.

    config t
    logging buffered [<sev lvl/name>]

The commands below allow you to toggle timestamps and sequence numbers. The `service timestamp` command allows you customize how much information to add as part of the timestamp. Note that the `localtime` option allows you to log to the buffer in the local time instead of UTC (the default).

    config t
    [no] service timestamps log datetime localtime show-timezone msec year
    [no] service sequence-numbers

The commands configure logging to syslog server

    logging console 7
    logging monitor debugging
    logging buffered 4
    logging host <ip addr>
    logging trap <sev lvl/name>

## <span id="ntp"></span>NTP

### Setting the time and time zone

    config t    
    clock timezone EST -5
    clock summer-time EDT recurring
    exit
    !
    clock set 20:50:49 21 October 2016

    show clock

When issuing the `show clock` command, you may see a symbol before the time:

* `00:20:14.301 EST Wed Apr 1 2019`: When the clock is synchronized or has been set manually, the time is displayed without a symbol
* `*00:20:14.301 EST Wed Apr 1 2019`: When the clock has never been synchronized nor set, the time is displayed with an asterisk.
* `.00:20:14.301 EST Wed Apr 1 2019`: When the clock is not synchronized but was synchronized in the past, a dot appears before the time.

### Implementing NTP clients, servers, and client/server mode
An NTP client/server plays both roles, being a server to a client and the client of another server. A client/server can have a lower stratum than its server so it will rely on its server when available and rely on its internal clock when it's not.

    config t
    ntp server <ip addr>  !-- can have multiple
    ntp master <stratum>  !-- 1-15; lower is better

    !-- NTP show commands

    show ntp status
    show ntp associations

### NTP using a loopback interface for better availability

    config t
      interface loopback 0
      ip address <addr> <mask>
      exit
    !
    ntp master
    ntp source loopback0

## <span id="cdp"></span>Cisco Discovery Protocol (CDP) and Link Layer Discovery Protocol (LLDP)

### CDP

    config t
    cdp run
    !
    int fa0/1
      no cdp enable

    !-- Show commands

    show cdp neighbors [<if>]
    show cdp neighbors [<if>] detail
    show cdp entry <hostname/FQDN>
    show cdp
    show cdp interface [<if>]
    show cdp traffic

### LLDP

    config t
    lldp run
    !
    int fa0/1
      no lldp transmit
      no lldp receive

    !-- Show commands

    show lldp neighbors
    show lldp entry <name>
        show lldp interface [<if>]