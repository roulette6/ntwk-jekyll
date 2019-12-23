---
title: "The OSI model"
layout: page
category: basics
order: 1
---

The __Open System Interconnection__ (OSI) model is a networking model that explains how communication across computer networks should work. Understanding this model is key to fixing issues. For example, if a device can't reach its gateway, you're already narrowing the scope of the problem to layers 1 - 3. The OSI model is something you will use any time you perform troubleshooting.

The table below shows the layers of the OSI networking model. The model is in reverse numerical order because each higher layer depends on the lower layer beneath it. For example, if you have a bad <acronym title="network interface card">NIC</acronym> or cable, the frame received by the network device may have errors that will cause it to be discarded.

| Layer | Name         | PDU                                               |
|:---       |:---          |:---                                               |
| 7         | Application  | Data (e.g., email client, web browser)            |
| 6         | Presentation | Data (e.g., images, word docs)                    |
| 5         | Session      | Data (manages sessions between client and server) |
| 4         | Transport    | Segments (e.g., TCP and UDP)                      |
| 3         | Network      | Packets (e.g., IPv4 and IPv6)                     |
| 2         | Data Link    | Frames                                            |
| 1         | Physical     | Bits                                              |