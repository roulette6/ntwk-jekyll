---
title: "Infrastructure Security"
layout: default
category: infrastructure
order: 15
---

## Securing IOS passwords
You can harden the enable secret by indicating the hash algorithm that should be used. The list of algorithms is below:

* 0: Plain text
* 7: Encrypted (easy to crack)
* 5: MD5
* Type 4: PBKDF2
* Type 8: PBKDF2 (sha-256)
* Type 9: Scrypt

Syntax

    enable [algorithm-type <algorithm>] secret <secret>

    enable algorithm-type md5 secret <secret>
    enable algorithm-type sha-256 secret <secret>
    enable algorithm-type scrypt secret <secret>

### Hiding password for local usernames
Note that a user cannot have both a password and a secret, so either assign a secret or assign a password and password encryption. If you have a hashed secret from a previous running-config, you can enter it as well (not shown below).

    username <user> secret <secret>

    !-- or

    username <user> password <password>
    service password-encryption

## ACLs for vty lines

    line vty 0 15
      access-class <ACL name/num> in

## Cisco device hardening

### Configure login banners
The banners are shown in different order depending on login method (console/SSH). SSHv1 doesn't show the login banner.

    banner [motd] + msg +
    banner login + msg +
    banner exec + msg +

#### Order in which banners are shown
* Console and telnet
    - MOTD
    - Login
    - [User logs in]
    - Exec
* SSH
    - Login
    - [User logs in]
    - MOTD
    - Exec

### Securing unused interfaces
* Shutdown unused interfaces
* Prevent trunking by hardcoding access ports
* Assign ports to unused VLAN
* Set the native VLAN on trunks to unused VLAN
