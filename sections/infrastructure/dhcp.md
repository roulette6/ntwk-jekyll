---
title: "Dynamic Host Configuration Protocol (DHCP)"
layout: default
category: infrastructure
order: 1
---

## DHCP for IPv4
DHCP, defined in [IETF RFC 2131](https://tools.ietf.org/html/rfc2131), allows clients to dynamically get an IP address, default gateway, and DNS servers. This is done with the following series of messages:

* __Discover:__ Broadcast by client
* __Offer:__ Broadcast by the DHCP server
* __Request:__ Broadcast so other DHCP servers will know the offer that was accepted. The request includes the offer so the DHCP servers know which offer was accepted when multiple offers are made.
* __Acknowledgment:__ Sent via unicast, although it can also be broadcast.

Clients can try to renew a lease so the IP address is not offered to someone else. Switches must be multilayer switches with an SVI in the VLAN of the requester. Switches automatically exclude addresses in use by SVIs and brodcast addresses.

### Configure a DHCP server

    config t
    ip dhcp excluded-address <start ip> [<end ip>]
    ip dhcp pool <name>
      network <prefix> <mask>
      default-router <ip addr> [<ip addr2] ...
      dns-server <ip addr> [<ip addr2>}...
      domain-name example.com
      lease {infinite | <days> [hrs] [mins]}
      exit

    !-- Monitor leases or clear (remove) bindings

    show ip dhcp binding
    clear ip dhcp binding {* | <ip addr>}

#### Configuring a manual address binding
You can configure manual address bindings on a DHCP server. You can identify a client by its client identifier, which tends to start with 01 for Ethernet, thereby offsetting the MAC address portion by two hex digits. If you have trouble figuring out the correct client identifier, you can debug ip dhcp server so the client request info is displayed.

        conf t
        ip dhcp pool my-pc
          host 192.168.1.99 255.255.255.0
          client-identifier 0100.50b6.5bc0.b5
        end
        debug ip dhcp server

#### Configuring DHCP options
Some devices need additional information, such as bootstrap info so it can find the address of a machine offering a needed service. You can provide this as DHCP options when you configure the DHCP server.

Common DHCP options (configured via `option <opt num> ip <addr>` in dhcp-config mode)

* __43:__ Wireless LAN controller for lightweight WAPs
* __69:__ Location of SMTP server
* __70:__ Location of a POP3 mail server
* __150:__ Location of a TFTP server for Cisco IP phones

#### DHCP relay
You can configure one more helper addresses on an SVI:

    int vlan5
      ip addr 192.168.1.1 255.255.255.0
      ip helper-address 192.168.199.4
      exit

***

## DHCP for IPv6

### DHCPv6
DHCPv6 doesn't allow you to exclude addresses nor configure manual bindings.

    config t
    ipv6 dhcp pool <name>
      address prefix <ipv6 prefix>
      dns-server <address>
      domain-name <name>
    !
    int <if>
      ipv6 address <addr>
      ipv6 dhcp server <pool name>
      no shut

### DHCPv6 lite
DHCPv6 Lite combines stateless autoconfig for address mgmt with DHCP option mgmt. You can configure DHCPv6 Lite by defining a DHCPv6 pool and omitting the address prefix command so clients won't use DHCPv6 to obtain their addresses.

    ipv6 dhcp pool v6-users
      dns-server <addr>
      domain-name jm.com
      exit
    !
    interface <if>
      ipv6 address <addr>
      ipv6 dhcp server v6-users
      ipv6 nd other-config-flag
      no shut

### Configuring a DHCP relay agent

    interface <if>
      ipv6 dhcp relay destination <ipv6 addr>
    
### Verifying IPv6 DHCP operation

    show ipv6 dhcp pool
    show ipv6 dhcp binding { * | <ipv6 addr>}