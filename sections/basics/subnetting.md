---
title: "Subnetting"
layout: page
category: basics
order: 4
---

IP addresses are 32-bit digits displayed in dotted decimal notation. All classful IP addresses can be divided into __network bits__ and __host bits__. For example, the IP address `192.168.1.1` has a network mask of `255.255.255.0`, which can also be written as `/24` in CIDR notation. The entire IP address written in CIDR notation is `192.168.1.1/24`. CIDR stands for Classless Inter-Domain Routing and indicates how many network bits there are in an IP address. In this case, there are 24.

The host bits of an IP address indicate the part of the IP address that cannot change because it denotes the network and the part that can change to denote the different hosts in that network. For example, class C networks all have 24 network bits, which means that the first three octets cannot change.

But what do you do if you're designing a network and find that you have 260 database servers that need to be in the same network? You can subdivide a larger network to make it an adequate size. You do this by taking some host bits and turning them into subnet bits so you can extend the network bits. For example, you can take 172.16.0.0/16 and subnet it into an equal number of networks that will accommodate 260 servers.

260 servers per subnet means a /24 subnet will be too small (256 addresses minus the network and broadcast IDs = 254 addresses). That means a /23 subnet will be required. This will fit 510 devices (512 minus subnet and broadcast IDs). This is way more IP addresses than we need, but because of the way subnetting works, you have to go from 256 to 512; there's no in-between.

Why can we only increase the number of hosts in a subnet by multiples of two? Because all the network bits must be contiguous, meaning they cannot be separated, and because we're counting in binary. Let's take for granted how binary works and let's instead enjoy the table below.

## Subnetting table

| CIDR | Network mask    | Wildcard mask   | Total IPs      |
| :--- | :---            | :---            | ---:           |
/32    | 255.255.255.255 | 0.0.0.0         | 1              |
/31    | 255.255.255.254 | 0.0.0.1         | 2              |
/30    | 255.255.255.252 | 0.0.0.3         | 4              |
/29    | 255.255.255.248 | 0.0.0.7         | 8              |
/28    | 255.255.255.240 | 0.0.0.15        | 16             |
/27    | 255.255.255.224 | 0.0.0.31        | 32             |
/26    | 255.255.255.192 | 0.0.0.63        | 64             |
/25    | 255.255.255.128 | 0.0.0.127       | 128            |
/24    | 255.255.255.0   | 0.0.0.255       | 256            |
/23    | 255.255.254.0   | 0.0.1.255       | 512            |
/22    | 255.255.252.0   | 0.0.3.255       | 1,024          |
/21    | 255.255.248.0   | 0.0.7.255       | 2,048          |
/20    | 255.255.240.0   | 0.0.15.255      | 4,096          |
/19    | 255.255.224.0   | 0.0.31.255      | 8,192          |
/18    | 255.255.192.0   | 0.0.63.255      | 16,384         |
/17    | 255.255.128.0   | 0.0.127.255     |  32,768        |
/16    | 255.255.0.0     | 0.0.255.255     |  65,536        |
/15    | 255.254.0.0     | 0.1.255.255     |  131,072       |
/14    | 255.252.0.0     | 0.3.255.255     |  262,144       |
/13    | 255.248.0.0     | 0.7.255.255     |  524,288       |
/12    | 255.240.0.0     | 0.15.255.255    |  1,048,576     |
/11    | 255.224.0.0     | 0.31.255.255    |  2,097,152     |
/10    | 255.192.0.0     | 0.63.255.255    |  4,194,304     |
/9     | 255.128.0.0     | 0.127.255.255   |  8,388,608     |
/8     | 255.0.0.0       | 0.255.255.255   |  16,777,216    |
/7     | 254.0.0.0       | 1.255.255.255   |  33,554,432    |
/6     | 252.0.0.0       | 3.255.255.255   |  67,108,864    |
/5     | 248.0.0.0       | 7.255.255.255   |  134,217,728   |
/4     | 240.0.0.0       | 15.255.255.255  |  268,435,456   |
/3     | 224.0.0.0       | 31.255.255.255  |  536,870,912   |
/2     | 192.0.0.0       | 63.255.255.255  |  1,073,741,824 |
/1     | 128.0.0.0       | 127.255.255.255 |  2,147,483,648 |
/0     | 0.0.0.0         | 255.255.255.255 |  4,294,967,296 |