---
title: "Access Control Lists"
layout: page
category: basics
order: 6
---

## Common protocols and their ports

| Protocol | Port      | Use          |
| :---     | :---      | :---         |
| TCP      | 20        | FTP data     |
| TCP      | 21        | FTP          |
| TCP      | 22        | SSH          |
| TCP      | 23        | Telnet       |
| TCP      | 25        | SMTP         |
| TCP      | 49        | TACACS       |
| TCP      | 53        | DNS server   |
| TCP      | 80        | HTTP         |
| TCP      | 110       | POP3         |
| TCP      | 143       | IMAP4        |
| TCP      | 443       | SSL/HTTPS    |
| TCP      | 1645,1646 | RADIUS (old) |
| TCP      | 1812,1813 | RADIUS (new) |
| UDP      | 53        | DNS client   |
| UDP      | 67,68     | DHCP         |
| UDP      | 69        | TFTP         |
| UDP      | 123       | NTP          |
| UDP      | 161       | SNMP         |
| UDP      | 514       | Syslog       |

## Types of ACLs
* Standard numbered ACLs (1-99; 1300-1999): Match source IP address
* Extended numbered ACLs (100-199; 2000-2699): Match source/destination IP address, port, and other parameters
* Named ACLs: Improved editing with sequence numbers

## Standard numbered ACLs

    access-list 1 permit host 192.168.1.25
    access-list 1 permit 192.168.2.0 0.0.0.255
    !
    ip access-list standard 1
      30 deny host 192.168.4.1
      end

## Extended numbered ACLs

    access-list 100 permit tcp 192.168.2.0 0.0.0.255 host 192.168.3.28 eq 22

## Named ACLs
The syntax below works with both named ACLs and numbered ACLs.

    ip access-list extended DENY-PVT
      10 deny ip 10.0.0.0 0.255.255.255 any
      20 deny ip 172.16.0.0 0.0.255.255 an
      no 20
      20 deny ip 172.16.0.0 0.0.31.255 any
      exit
      !
    ip access-list extended 101
      10 permit tcp host 192.168.1.1 gt 1023 host 192.168.2.1 eq 22
      20 permit icmp any any
      30 deny ip any any

## Applying ACLs

    int fa0/1
      ip access-group 1 in
    !
    line vty 0 15
      access-class 100 in

## Resequencing ACLs

    ip access-list resequence < ACL name/num> <start> <step>
    ip access-list resequence 101 5 5

## Time-based ACLs
Time ranges can be absolute (with a start and end date) or periodic (recurring).

    config t
    time-range WEEKDAYS
      periodic weekdays 06:00 to 18:00
      exit
    !
    time-range VACATION
      absolute start 18:00 31 Mar 2017 end 06:00 15 Apr 2017
      exit
    !
    access-list 100 permit tcp any host 192.168.1.10 eq 22 time-range WEEKDAYS
    access-list 101 deny tcp any host 192.168.2.15 eq 22 time-range VACATION

Options for periodic ACLs

    periodic ?
      Friday     Friday
      Monday     Monday
      Saturday   Saturday
      Sunday     Sunday
      Thursday   Thursday
      Tuesday    Tuesday
      Wednesday  Wednesday
      daily      Every day of the week
      weekdays   Monday thru Friday
      weekend    Saturday and Sunday

## Infrastructure ACLs
Infrastructure ACLs are typically extended ACLs applied inbound to edge routers to prevent malicious traffic from entering.

    ip access-list extended INFRASTRUCTURE
      deny icmp any any fragments
      deny ip any any fragments
      permit tcp host <ext bgp peer> host <int bgp peer> eq bgp
      permit tcp host <ext bgp peer> eq bgp host <int bgp peer>
      !-- add rules to deny inbound traffic to internal networks
      permit ip any any
      !
      int gi0/0/1
        ip access-group INFRASTRUCTURE in