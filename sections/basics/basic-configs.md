---
title: "Basic configs"
layout: page
category: basics
order: 9
---

## Useful lab commands

### Exec mode

    show history
    terminal history size <num> !-- exec mode; current session

### Config mode

    history size <num>
    no logging console          !-- disable log msgs from console
    logging synchronous         !-- sync syslog msgs with command msgs
    no ip domain-lookup
    !
    line con 0
      exec-timeout <min> <sec>

***

## Line passwords

    conf t
    enable secret <secret>
    !
    line console 0
      password <password>
      login
      exit
    !
    line vty 0 4
      password <password>
      login
    end

## Local passwords

    conf t
    enable secret <secret>
    !
    username josue [priv 15] secret cisco
    line console 0
      login local
    !
    line vty 0 4
      login local
    end

***

## Configuring SSH

    conf t
    username josue priv 15 secret cisco
    !
    hostname rtr1
    ip domain-name example.com
    !
    crypto key generate rsa modulus 1024
    ip ssh version 2
    !
    line vty 0 15
      transport input ssh
      login local

    !-- Verification

    show ip ssh
    show ssh

***

## Banners
There are three kinds of banners. The `+` is a delimiter and can be any character that does not appear in the banner.

1. __MOTD:__ Shown before login prompt for messages that change<br/>`banner motd +msg+`
2. __Login:__ Shown before login prompt but after MOTD for permanent messages<br/>`banner motd +msg+`
3. __Exec:__ Shown after login prompt to authenticated users<br/>`banner motd +msg+`