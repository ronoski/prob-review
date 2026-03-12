# Networking Fundamentals

> A complete reference structured around the 4 TCP/IP layers — covering OSI as a reference model, then deep-diving each layer from the bottom up.

---

## Table of Contents

1. [Why Network Models Exist](#1-why-network-models-exist)
2. [OSI Model — Reference](#2-osi-model--reference)
3. [TCP/IP Model — Overview](#3-tcpip-model--overview)
4. [Layer 1 — Network Access Layer](#4-layer-1--network-access-layer)
5. [Layer 2 — Internet Layer](#5-layer-2--internet-layer)
6. [Layer 3 — Transport Layer](#6-layer-3--transport-layer)
7. [Layer 4 — Application Layer](#7-layer-4--application-layer)
8. [Data Encapsulation & the Full Journey](#8-data-encapsulation--the-full-journey)
9. [NAT — Network Address Translation](#9-nat--network-address-translation)
10. [Key Concepts at a Glance](#10-key-concepts-at-a-glance)

---

## 1. Why Network Models Exist

Modern networks are extremely complex. Different vendors build hardware and software that must interoperate. To solve this, networking is divided into **layers**, where each layer has a specific, well-defined responsibility.

This layered architecture provides:

- **Standardization** — vendors can build compatible systems
- **Modularity** — changes in one layer do not affect others
- **Troubleshooting simplicity** — problems can be isolated by layer
- **Interoperability** — devices from different manufacturers communicate

Two important models:
1. **OSI Model** — theoretical 7-layer model (ISO standard), used as a reference
2. **TCP/IP Model** — practical 4-layer model used by the real Internet

---

## 2. OSI Model — Reference

The **OSI (Open Systems Interconnection)** model divides network communication into **7 layers**. Data travels **down** the layers on the sender side and **up** on the receiver side.

```
  Sender                                        Receiver
+------------------+                        +------------------+
| 7. Application   | <-- HTTP, FTP, DNS --> | 7. Application   |
+------------------+                        +------------------+
| 6. Presentation  | <-- TLS, JPEG, JSON -> | 6. Presentation  |
+------------------+                        +------------------+
| 5. Session       | <-- NetBIOS, RPC    -> | 5. Session       |
+------------------+                        +------------------+
| 4. Transport     | <-- TCP, UDP        -> | 4. Transport     |
+------------------+                        +------------------+
| 3. Network       | <-- IP, ICMP        -> | 3. Network       |
+------------------+                        +------------------+
| 2. Data Link     | <-- Ethernet, MAC   -> | 2. Data Link     |
+------------------+                        +------------------+
| 1. Physical      | <-- Cables, Signals -> | 1. Physical      |
+------------------+                        +------------------+
        |                                           ^
        |           actual bits on wire             |
        +-------------------------------------------+
```

### Quick Reference Table

| # | Layer        | PDU     | Key Devices        | Protocols              |
|---|--------------|---------|--------------------|------------------------|
| 7 | Application  | Data    | —                  | HTTP, DNS, FTP, SMTP   |
| 6 | Presentation | Data    | —                  | TLS, JPEG, ASCII       |
| 5 | Session      | Data    | —                  | NetBIOS, RPC           |
| 4 | Transport    | Segment | —                  | TCP, UDP               |
| 3 | Network      | Packet  | Router             | IP, ICMP, BGP          |
| 2 | Data Link    | Frame   | Switch, Bridge     | Ethernet, ARP          |
| 1 | Physical     | Bit     | Hub, Cable, NIC    | DSL, 802.11 (Wi-Fi)    |

### Memory Aid

> **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing *(top → bottom, 7→1)*

> **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way *(bottom → top, 1→7)*

---

## 3. TCP/IP Model — Overview

TCP/IP is the **practical protocol suite** that powers the modern Internet. It collapses the 7 OSI layers into **4 layers**.

```
  TCP/IP Layer        OSI Equivalent       Protocols
  ┌─────────────────┐
  │ 4. Application  │ OSI 5 + 6 + 7       HTTP, HTTPS, DNS, FTP, SMTP, SSH
  ├─────────────────┤
  │ 3. Transport    │ OSI 4               TCP, UDP
  ├─────────────────┤
  │ 2. Internet     │ OSI 3               IP (v4/v6), ICMP, ARP
  ├─────────────────┤
  │ 1. Network      │ OSI 1 + 2           Ethernet, Wi-Fi, MAC
  │    Access       │
  └─────────────────┘
```

### OSI vs TCP/IP Mapping

```
  OSI Model              TCP/IP Model
  ─────────────────────────────────────────────
  7 Application  ┐
  6 Presentation ├──────── 4. Application
  5 Session      ┘

  4 Transport    ──────── 3. Transport

  3 Network      ──────── 2. Internet

  2 Data Link    ┐
  1 Physical     ┘──────── 1. Network Access
```

| Feature            | OSI        | TCP/IP                  |
|--------------------|------------|-------------------------|
| Layers             | 7          | 4                       |
| Developed by       | ISO        | DARPA                   |
| Usage              | Conceptual | Real-world Internet     |
| Presentation Layer | Separate   | Merged into Application |
| Session Layer      | Separate   | Merged into Application |

---

## 4. Layer 1 — Network Access Layer

> **OSI equivalent:** Physical (L1) + Data Link (L2)
> **Job:** Deliver frames between devices on the **same local network** using hardware addresses, and transmit raw bits over the physical medium.

```
  ┌─────────────────┐
  │ 4. Application  │
  ├─────────────────┤
  │ 3. Transport    │
  ├─────────────────┤
  │ 2. Internet     │
  ├─────────────────┤
  │ 1. Network      │  ◄── YOU ARE HERE
  │    Access       │
  └─────────────────┘
```

### Physical Sub-layer

Transmits raw **bits (0s and 1s)** over a physical medium. No awareness of addresses or structure — just signals.

```
  Binary data:   1 0 1 1 0 0 1 0 1 0 ...
                        │
                        ▼
  Medium:    ──[copper wire]──  or  ~~[radio wave]~~  or  ══[fiber optic]══
                        │
                        ▼
  Received:  1 0 1 1 0 0 1 0 1 0 ...
```

| Medium         | Technology          | Speed (typical)  |
|----------------|---------------------|------------------|
| Copper cable   | Ethernet (Cat5e/6)  | 1–10 Gbps        |
| Fiber optic    | Single/multi-mode   | 10–400 Gbps      |
| Radio waves    | Wi-Fi (802.11)      | 54 Mbps–9.6 Gbps |

Devices at this sub-layer: **Hubs, Repeaters, Network cables, NICs (physical)**

---

### Data Link Sub-layer

Wraps IP packets into **frames** for delivery across a single network hop, using **MAC addresses** for local identification.

#### Ethernet Frame Structure

```
  ┌──────────┬──────────┬──────────┬──────────────────────┬───────┐
  │ Dst MAC  │ Src MAC  │EtherType │       PAYLOAD        │  FCS  │
  │ 6 bytes  │ 6 bytes  │ 2 bytes  │    (IP Packet)       │4 bytes│
  └──────────┴──────────┴──────────┴──────────────────────┴───────┘

  Dst MAC   — MAC address of next hop (could be a router, not final dst)
  Src MAC   — MAC address of sending NIC
  EtherType — 0x0800 = IPv4, 0x86DD = IPv6, 0x0806 = ARP
  FCS       — Frame Check Sequence: CRC error detection trailer
```

#### MAC Address

A **Media Access Control (MAC) address** is a 48-bit hardware address burned into every NIC, written as six hex pairs.

```
  00 : 1A : 2B : 3C : 4D : 5E
  └─────────────┘   └─────────┘
   OUI (vendor ID)   Device ID
   (first 3 bytes)   (last 3 bytes)

  - Unique per physical interface (in theory)
  - Used only within a local network segment
  - Routers rewrite MAC addresses at every hop (IPs stay the same)
```

#### ARP — Address Resolution Protocol

IP delivers to machines by IP address, but the local network needs a MAC address. **ARP bridges this gap**.

```
  Scenario: Your PC (192.168.1.10) wants to send to Router (192.168.1.1)
            but doesn't know the router's MAC address.

  Step 1 — ARP Request (broadcast to all devices on LAN):
  ┌──────────────────────────────────────────────────────────┐
  │ Src MAC: AA:BB:CC:11:22:33  Src IP: 192.168.1.10        │
  │ Dst MAC: FF:FF:FF:FF:FF:FF  Dst IP: 192.168.1.1         │
  │ "Who has 192.168.1.1? Tell 192.168.1.10"                │
  └──────────────────────────────────────────────────────────┘
         │
         │  broadcast — every device on LAN receives this
         ▼
  Step 2 — ARP Reply (unicast from router):
  ┌──────────────────────────────────────────────────────────┐
  │ Src MAC: DD:EE:FF:44:55:66  Src IP: 192.168.1.1         │
  │ Dst MAC: AA:BB:CC:11:22:33  Dst IP: 192.168.1.10        │
  │ "192.168.1.1 is at DD:EE:FF:44:55:66"                   │
  └──────────────────────────────────────────────────────────┘

  Step 3 — Your PC caches this in its ARP table:
  192.168.1.1  →  DD:EE:FF:44:55:66   (expires after ~20 min)

  Now frames can be sent to the correct MAC address.
```

#### What Switches Do

```
  Switch operates at the Data Link layer.
  It learns MAC addresses from incoming frames and builds a MAC table.

  Device A (AA:AA) ──┐                    ┌── Device C (CC:CC)
  Device B (BB:BB) ──┼── [SWITCH] ────────┤
                     │                    └── Device D (DD:DD)
                     │
  MAC Table:
  ┌──────────────────┬──────┐
  │ MAC Address      │ Port │
  ├──────────────────┼──────┤
  │ AA:AA:AA:AA:AA:AA│  1   │
  │ BB:BB:BB:BB:BB:BB│  2   │
  │ CC:CC:CC:CC:CC:CC│  3   │
  └──────────────────┴──────┘
  Frame for CC:CC → sent out Port 3 only (not flooded to all)
```

Devices at this sub-layer: **Switches, Bridges, NICs (logical)**

---

## 5. Layer 2 — Internet Layer

> **OSI equivalent:** Network (L3)
> **Job:** Route packets across **multiple networks** from source to destination using logical IP addresses.

```
  ┌─────────────────┐
  │ 4. Application  │
  ├─────────────────┤
  │ 3. Transport    │
  ├─────────────────┤
  │ 2. Internet     │  ◄── YOU ARE HERE
  ├─────────────────┤
  │ 1. Network      │
  │    Access       │
  └─────────────────┘
```

Key protocols: **IP (IPv4, IPv6), ICMP**

---

### Internet Protocol (IP)

IP is **connectionless** and **best-effort** — each packet is routed independently with no guarantee of delivery, order, or integrity. Upper layers (TCP) add reliability when needed.

```
  [Host A] ──packet──► [Router 1] ──packet──► [Router 2] ──packet──► [Host B]

  Each router reads the Destination IP and makes an independent
  forwarding decision. Packets may take different paths.
```

---

### IPv4

#### Address Format

IPv4 uses a **32-bit** address, written as four decimal octets.

```
  192  .  168  .   1   .  100
  11000000  10101000  00000001  01100100
  ←8 bits→  ←8 bits→  ←8 bits→  ←8 bits→
                = 32 bits total (~4.3 billion addresses)
```

#### Network vs Host Portion

```
  IP Address:   192.168.  1.100
  Subnet Mask:  255.255.255.  0
                |___________|  |___|
                Network part   Host part

  Network:   192.168.1.0    (identifies the network)
  Host:      .100           (identifies the device)
  Broadcast: 192.168.1.255  (all devices on network)
```

#### CIDR Notation

Subnet masks are written as a **prefix length** — number of bits reserved for the network.

| CIDR | Subnet Mask      | Usable Hosts | Example Use    |
|------|------------------|--------------|----------------|
| /8   | 255.0.0.0        | ~16.7M       | Large org      |
| /16  | 255.255.0.0      | 65,534       | Campus network |
| /24  | 255.255.255.0    | 254          | Home / office  |
| /28  | 255.255.255.240  | 14           | Small subnet   |
| /30  | 255.255.255.252  | 2            | Point-to-point |
| /32  | 255.255.255.255  | 1            | Single host    |

#### Special / Reserved IPv4 Addresses

| Address / Range    | Purpose                                 |
|--------------------|-----------------------------------------|
| 127.0.0.1          | Loopback — localhost                    |
| 0.0.0.0            | Unspecified / any address               |
| 10.0.0.0/8         | Private — large enterprise (NAT)        |
| 172.16.0.0/12      | Private — enterprise (NAT)              |
| 192.168.0.0/16     | Private — home/office (NAT)             |
| 169.254.0.0/16     | Link-local / APIPA (no DHCP found)      |
| 224.0.0.0/4        | Multicast                               |
| 255.255.255.255    | Limited broadcast                       |

#### IPv4 Packet Structure

```
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |Version|  IHL  |    DSCP   |ECN|         Total Length          |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |         Identification        |Flags|     Fragment Offset      |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |  Time to Live |    Protocol   |        Header Checksum        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                       Source IP Address                       |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                    Destination IP Address                     |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         DATA ...                              |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  Key fields:
  TTL      — Decremented at each router; dropped at 0
  Protocol — 6=TCP, 17=UDP, 1=ICMP
  Src/Dst  — 32-bit addresses
```

#### TTL — Time To Live

```
  Packet starts with TTL = 64 (typical default)

  [Your PC] TTL=64 ──► [Router 1] TTL=63 ──► [Router 2] TTL=62 ──► ...

  If TTL reaches 0:
    - Router DROPS the packet
    - Sends ICMP "Time Exceeded" message back to sender
    - Prevents packets from looping forever in the network

  traceroute exploits this:
    Send TTL=1 → Router 1 drops it, reveals its IP
    Send TTL=2 → Router 2 drops it, reveals its IP
    ...until destination is reached (path is mapped)
```

#### How Routing Works

```
  Your PC                                          Web Server
  192.168.1.5                                      93.184.216.34
      │
      │ (local network — same subnet)
      ▼
  Home Router ──► ISP Router ──► Internet ──► Destination
  192.168.1.1    203.0.113.1       ...         93.184.216.34

  Each router:
  1. Strips the incoming Ethernet frame
  2. Reads the Destination IP in the IP packet
  3. Looks up the next hop in its routing table
  4. Wraps the packet in a NEW frame (new src/dst MAC)
  5. Forwards out the correct interface

  IP addresses stay the same end-to-end.
  MAC addresses change at every hop.
```

---

### IPv6

#### Why IPv6?

```
  IPv4 address space: 2^32  =           4,294,967,296   (~4.3 billion)
  IPv6 address space: 2^128 = 340,282,366,920,938,463,463,374,607,431,768,211,456
                            = 340 undecillion

  Enough for every grain of sand on Earth — millions of times over.
```

#### Address Format

IPv6 uses **128 bits**, written as eight groups of four hex digits separated by colons.

```
  Full:      2001:0db8:85a3:0000:0000:8a2e:0370:7334

  Shortening rules:
  1. Drop leading zeros within each group:
     2001:db8:85a3:0:0:8a2e:370:7334

  2. Replace one run of consecutive all-zero groups with :::
     2001:db8:85a3::8a2e:370:7334
     (:: can appear only ONCE per address)

  Special:
  Loopback:  ::1          (equivalent to 127.0.0.1)
  All zeros: ::           (equivalent to 0.0.0.0)
```

#### IPv6 Address Types

| Type           | Prefix      | Description                            |
|----------------|-------------|----------------------------------------|
| Global Unicast | 2000::/3    | Public routable (like IPv4 public IPs) |
| Link-Local     | fe80::/10   | Same link only, auto-configured        |
| Loopback       | ::1/128     | Localhost                              |
| Multicast      | ff00::/8    | One-to-many (replaces broadcast)       |
| Anycast        | (unicast)   | Routed to nearest node in a group      |

#### IPv6 Packet Structure

Simpler than IPv4 — **fixed 40-byte header**, no checksum, no router-side fragmentation.

```
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |Version| Traffic Class |             Flow Label                |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |         Payload Length        |  Next Header  |   Hop Limit   |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                                                               |
  |                    Source Address (128 bits)                  |
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                                                               |
  |                 Destination Address (128 bits)                |
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  Hop Limit  = IPv6 equivalent of TTL
  Next Header = replaces Protocol field (supports extension headers)
  No Checksum (handled by upper layers)
  No Fragmentation at routers (hosts must fragment before sending)
```

---

### IPv4 vs IPv6 — Full Comparison

| Feature         | IPv4                   | IPv6                     |
|-----------------|------------------------|--------------------------|
| Address size    | 32 bits                | 128 bits                 |
| Address space   | ~4.3 billion           | 340 undecillion          |
| Notation        | Decimal (192.168.1.1)  | Hex colon (2001:db8::1)  |
| Header size     | 20–60 bytes (variable) | 40 bytes (fixed)         |
| Header checksum | Yes                    | No                       |
| Fragmentation   | Routers & hosts        | Hosts only               |
| Broadcast       | Yes                    | No (uses multicast)      |
| NAT required    | Usually                | No (all addresses public)|
| Auto-config     | DHCP                   | SLAAC or DHCPv6          |
| Loopback        | 127.0.0.1              | ::1                      |
| Security        | Optional (IPSec)       | Native (IPSec built-in)  |

---

### ICMP — Internet Control Message Protocol

ICMP is a **Layer 3 (Internet layer) helper protocol** used for **error messages and diagnostics**. It does not carry application data — it reports problems and enables network tools.

```
  ICMP sits alongside IP at the Internet layer:

  ┌─────────────┐
  │ Application │
  ├─────────────┤
  │  Transport  │  (TCP / UDP)
  ├──────┬──────┤
  │  IP  │ ICMP │  ← same layer; ICMP rides inside IP packets
  ├─────────────┤
  │  Data Link  │
  └─────────────┘
```

#### Common ICMP Message Types

| Type | Code | Meaning                      | Tool / Trigger  |
|------|------|------------------------------|-----------------|
| 8    | 0    | Echo Request                 | ping            |
| 0    | 0    | Echo Reply                   | ping response   |
| 3    | 0    | Dest Unreachable: Network    | no route        |
| 3    | 1    | Dest Unreachable: Host       | host down       |
| 3    | 3    | Dest Unreachable: Port       | port closed     |
| 11   | 0    | Time Exceeded (TTL = 0)      | traceroute      |

#### ping — Echo Request / Echo Reply

```
  Your PC                                   Remote Host
     │                                           │
     │── ICMP Echo Request (Type 8) ────────────►│
     │◄─ ICMP Echo Reply   (Type 0) ─────────────│
     │                                           │
     Round Trip Time (RTT) measured

  $ ping google.com
  64 bytes from 142.250.80.46: icmp_seq=1 ttl=117 time=12.4 ms
  64 bytes from 142.250.80.46: icmp_seq=2 ttl=117 time=11.9 ms

  icmp_seq — detects dropped packets
  ttl      — remaining TTL when reply arrived (reveals hop count)
  time     — round-trip time in milliseconds
```

#### traceroute — Mapping the Path via TTL

```
  TTL=1 ──► Router 1 drops it ──► sends ICMP Time Exceeded back ──► reveals IP
  TTL=2 ──► Router 2 drops it ──► sends ICMP Time Exceeded back ──► reveals IP
  TTL=3 ──► Destination reached  ──► sends Echo Reply

  Your PC          Router 1          Router 2          Google
    │                 │                 │                 │
    │── TTL=1 ───────►│                 │                 │
    │◄─ Time Exceeded ─│                 │                 │
    │                 │                 │                 │
    │── TTL=2 ────────────────────────►│                 │
    │◄─ Time Exceeded ──────────────────│                 │
    │                 │                 │                 │
    │── TTL=3 ─────────────────────────────────────────►│
    │◄─ Echo Reply ──────────────────────────────────────│

  Output:
  1   192.168.1.1      1.2 ms   (your home router)
  2   10.20.30.1       8.4 ms   (ISP gateway)
  3   203.0.113.1     10.1 ms   (ISP backbone)
  4   142.250.80.46   12.4 ms   (Google)
```

#### ICMPv6 — ICMP for IPv6

ICMPv6 is more critical than ICMPv4. It **replaces ARP** entirely via NDP (Neighbor Discovery Protocol).

| Function               | Type | Description                       |
|------------------------|------|-----------------------------------|
| Echo Request           | 128  | ping                              |
| Echo Reply             | 129  | ping reply                        |
| Destination Unreachable| 1    | Packet cannot be delivered        |
| Packet Too Big         | 2    | MTU exceeded                      |
| Time Exceeded          | 3    | Hop Limit = 0                     |
| Neighbor Solicitation  | 135  | Replaces ARP Request              |
| Neighbor Advertisement | 136  | Replaces ARP Reply                |
| Router Solicitation    | 133  | Host asks: "Where's the router?"  |
| Router Advertisement   | 134  | Router announces itself + prefix  |

```
  IPv4: "Who has 192.168.1.1?" → ARP broadcast (Layer 2)
  IPv6: "Who has fe80::1?"     → ICMPv6 Neighbor Solicitation (multicast)
```

---

## 6. Layer 3 — Transport Layer

> **OSI equivalent:** Transport (L4)
> **Job:** Deliver data to the right **application** on a machine (IP got the packet to the right machine; transport gets it to the right process).

```
  ┌─────────────────┐
  │ 4. Application  │
  ├─────────────────┤
  │ 3. Transport    │  ◄── YOU ARE HERE
  ├─────────────────┤
  │ 2. Internet     │
  ├─────────────────┤
  │ 1. Network      │
  │    Access       │
  └─────────────────┘
```

```
  Network Layer (IP):        Gets packet to the right MACHINE
  Transport Layer (TCP/UDP): Gets data to the right APPLICATION

  [Your PC: IP 192.168.1.10]
    ├─ Browser   ──► port 443  ──► Web Server
    ├─ Music App ──► port 443  ──► Spotify      (different 4-tuple)
    ├─ Game      ──► port 3074 ──► Game Server
    └─ SSH       ──► port 22   ──► SSH Daemon
```

Key protocols: **TCP, UDP**

---

### Ports

A **port** is a 16-bit number (0–65535) that identifies a specific process or service on a host.

```
  Socket   = IP Address + Port
  4-tuple  = (src IP, src port, dst IP, dst port)
             uniquely identifies every active connection on the Internet

  Example:
  192.168.1.10 : 54321   ◄──►   93.184.216.34 : 443
  └──────────────────┘           └──────────────────┘
      Your browser                  Web server (HTTPS)
```

#### Port Ranges

| Range         | Name       | Usage                                      |
|---------------|------------|--------------------------------------------|
| 0 – 1023      | Well-known | Reserved standard services (root to bind)  |
| 1024 – 49151  | Registered | Vendor/application-specific                |
| 49152 – 65535 | Ephemeral  | Temporary client ports assigned by OS      |

#### Common Well-Known Ports

| Port   | Protocol  | Service                  |
|--------|-----------|--------------------------|
| 20/21  | TCP       | FTP (data / control)     |
| 22     | TCP       | SSH                      |
| 23     | TCP       | Telnet (insecure)        |
| 25     | TCP       | SMTP (send email)        |
| 53     | TCP/UDP   | DNS                      |
| 67/68  | UDP       | DHCP (server/client)     |
| 80     | TCP       | HTTP                     |
| 110    | TCP       | POP3 (retrieve email)    |
| 143    | TCP       | IMAP (retrieve email)    |
| 443    | TCP       | HTTPS                    |
| 3306   | TCP       | MySQL                    |
| 5432   | TCP       | PostgreSQL               |
| 6379   | TCP       | Redis                    |

---

### Multiplexing & Demultiplexing

```
  MULTIPLEXING (sending side):
  Many apps share one IP layer — each labeled with port numbers.

  [HTTP app]  ─────┐
  [SSH app]   ─────┼──► Transport Layer ──► IP Layer ──► Network
  [DNS app]   ─────┘
              (adds src/dst port to each chunk of data)

  DEMULTIPLEXING (receiving side):
  Incoming data split to the correct app by reading the destination port.

  Network ──► IP Layer ──► Transport Layer ──┬──► [HTTP app]   (port 443)
                                             ├──► [SSH app]    (port 22)
                                             └──► [DNS app]    (port 53)
```

---

### TCP — Transmission Control Protocol

TCP provides **reliable, ordered, connection-oriented** delivery.

```
  ┌───────────────────────────────────┐
  │ Connection-oriented  (handshake)  │
  │ Reliable delivery    (retransmit) │
  │ Ordered              (seq nums)   │
  │ Error checked        (checksum)   │
  │ Flow controlled      (window)     │
  │ Congestion controlled(CWND)       │
  └───────────────────────────────────┘
```

#### TCP Segment Structure

```
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |          Source Port          |       Destination Port        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                        Sequence Number                        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                    Acknowledgment Number                      |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  | Offset | Reserved  |U|A|P|R|S|F|         Window Size         |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |           Checksum            |         Urgent Pointer        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                             DATA                              |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  Key flags:
  SYN — Synchronize sequence numbers (connection setup)
  ACK — Acknowledgment field is valid
  FIN — No more data from sender (teardown)
  RST — Reset / abort connection immediately
  PSH — Push data to application without buffering
```

---

#### TCP Connection Lifecycle

##### Step 1 — Three-Way Handshake (Setup)

Before any data flows, both sides synchronize their state.

```
  Client States                     Server States
  ┌──────────┐                      ┌──────────┐
  │  CLOSED  │                      │  CLOSED  │
  └────┬─────┘                      └────┬─────┘
       │ app calls connect()             │ app calls listen()
       │                           ┌────▼─────┐
       │                           │  LISTEN  │
  ┌────▼─────┐                     └────┬─────┘
  │ SYN_SENT │──── SYN ───────────────►│
  └────┬─────┘                    SYN_RECEIVED
       │◄─── SYN-ACK ──────────────────│
  ┌────▼─────┐──── ACK ───────────────►┌────▼─────┐
  │ESTABLISHED│                        │ESTABLISHED│
  └──────────┘                         └──────────┘
```

```
  Client                                          Server
  CLOSED                                          LISTEN
    │                                               │
    │  Step 1: SYN  (seq=1000, SYN=1, ACK=0)       │
    │──────────────────────────────────────────────►│
    │  "I want to connect. My seq starts at 1000."  │
    │                                               │
  SYN_SENT                                    SYN_RECEIVED
    │                                               │
    │  Step 2: SYN-ACK  (seq=5000, ack=1001)       │
    │◄──────────────────────────────────────────────│
    │  "My seq is 5000. Got 1000, expecting 1001."  │
    │                                               │
    │  Step 3: ACK  (seq=1001, ack=5001)            │
    │──────────────────────────────────────────────►│
    │  "Got 5000, expecting 5001. Ready."           │
    │                                               │
  ESTABLISHED                                 ESTABLISHED
    │════════════════ DATA ════════════════════════►│
```

**Why 3 steps?**
- SYN: Client picks its Initial Sequence Number (ISN)
- SYN-ACK: Server picks its ISN AND acknowledges client's ISN
- ACK: Client acknowledges server's ISN
Both sides must confirm each other's starting sequence number.

**Why random ISNs?**
```
  Predictable ISNs → attacker can forge packets with correct seq numbers
                   → session hijacking / blind injection attack
  Random ISNs     → computationally infeasible to guess
```

---

##### Step 2 — Data Transfer with Acknowledgment

```
  Client ──── Segment (seq=1001, len=500) ─────────────────────► Server
  Client ──── Segment (seq=1501, len=500) ─────────────────────► Server
  Client ◄─── ACK (ack=2001) ───────────────────────────────── Server
              "Got everything up to 2001, send from there"

  On loss: no ACK received within timeout → retransmit
  3 duplicate ACKs (same ack#) → fast retransmit (no timeout wait)
```

**Retransmission on loss:**
```
  Client ──── seq=1001 ────────────────────────────────────────► (delivered)
  Client ──── seq=1501 ─────────────X LOST
  Client ──── seq=2001 ────────────────────────────────────────► (delivered, gap)
  Client ◄─── ACK 1501 ──────────────────────────────────────── (still waiting)
  Client ◄─── ACK 1501  (duplicate)
  Client ◄─── ACK 1501  (3rd duplicate = FAST RETRANSMIT!)
  Client ──── seq=1501 (RETRANSMIT) ──────────────────────────►
  Client ◄─── ACK 2501 ────────────────────────────────────────
```

**Sequence numbers ensure correct ordering:**
```
  Segment 1: seq=1,  "Hello"   (bytes 1–5)
  Segment 2: seq=6,  " Worl"   (bytes 6–10)
  Segment 3: seq=11, "d"       (byte 11)

  Even if segments arrive as 3, 1, 2 — receiver reorders by seq number.
  Result: "Hello World"
```

---

##### Step 3 — Flow Control (Sliding Window)

Prevents a fast sender from overwhelming a slow receiver.

```
  Receiver advertises its free buffer space in the Window Size field.

  Window = 10000 bytes free:
  [__________] → sender can transmit up to 10000 bytes before needing ACK

  Sender sends 10000 bytes:
  [##########] → Window = 0 → STOP!

  App reads 5000 bytes from buffer:
  [#####_____] → Window = 5000 → GO! Send more.

  Timeline:
  Sender                          Receiver
    │── 3000 bytes (win=10000) ──►│  used: 3000
    │── 3000 bytes ──────────────►│  used: 6000
    │── 4000 bytes ──────────────►│  used: 10000 (full)
    │   WAIT...                   │
    │◄── ACK, window=5000 ────────│  app consumed 5000
    │── 5000 bytes ──────────────►│
```

---

##### Step 4 — Congestion Control

Handles **network** congestion (separate from receiver buffer flow control).

```
  TCP maintains a Congestion Window (CWND).
  Actual send rate = min(Window Size, CWND)

  Phase 1 — Slow Start:
  CWND starts at 1 MSS, doubles every RTT (exponential growth)

  CWND
  ^
  16|              *
   8|          *
   4|      *
   2|  *
   1|*
  --+──────────► RTTs

  Phase 2 — Congestion Avoidance:
  CWND grows +1 MSS per RTT (linear) after reaching threshold

  Phase 3 — Loss Detected:
  CWND drops sharply → restart from slow start or halve

  Algorithms: TCP Tahoe, Reno, CUBIC (Linux default), BBR (Google)
```

---

##### Step 5 — Four-Way Handshake (Teardown)

Each direction closes independently — that's why it takes 4 steps.

```
  Client                                          Server
  ESTABLISHED                                 ESTABLISHED
    │                                               │
    │  Step 1: FIN (seq=2000)                       │
    │──────────────────────────────────────────────►│  "I'm done sending"
    │                                               │
  FIN_WAIT_1                                  CLOSE_WAIT
    │                                               │
    │  Step 2: ACK (ack=2001)                       │
    │◄──────────────────────────────────────────────│  "Got your FIN"
    │                                               │  (may still send data)
  FIN_WAIT_2                                        │
    │◄══════════════ DATA (remaining) ══════════════│
    │                                               │
    │  Step 3: FIN (seq=8000)                       │
    │◄──────────────────────────────────────────────│  "I'm done too"
    │                                               │
  TIME_WAIT                                   LAST_ACK
    │                                               │
    │  Step 4: ACK (ack=8001)                       │
    │──────────────────────────────────────────────►│
    │                                               │
    │  wait 2×MSL (~60–120s)                   CLOSED
    │
  CLOSED

  TIME_WAIT ensures: (1) final ACK reaches server, (2) old duplicate
  packets from this connection cannot corrupt a new connection.
```

##### RST — Abrupt Reset

```
  Client ──── SYN port=9999 ──────────────────────────────────► Server
  Client ◄─── RST ─────────────────────────────────────────── Server
  "Nothing is listening on port 9999"

  RST is also sent when:
  - Packet arrives for a non-existent connection
  - Application crashes without closing sockets
  - A firewall rejects the connection
```

---

#### TCP Interview Questions

**Q: Why 3 steps for setup but 4 for teardown?**
> SYN and SYN-ACK can be combined — server both acknowledges and initiates in one segment. But FIN and ACK cannot be combined at teardown: server ACKs the client's FIN immediately, but may still have data to send before it can send its own FIN.

**Q: Can the handshake be done in 2 steps?**
> No. Without the third ACK, the server doesn't know its SYN-ACK arrived. It would allocate resources without confirmation — vulnerable to SYN flood attacks.

**Q: What is a SYN flood attack?**
```
  Attacker sends thousands of SYNs with spoofed source IPs.
  Server sends SYN-ACK and allocates state — ACK never comes.
  Half-open connection table fills → legitimate connections refused (DoS).

  Mitigation: SYN cookies — server encodes state into its ISN,
  so no memory is allocated until the final ACK is verified.
```

**Q: What is TIME_WAIT and why does it matter?**
```
  After the final ACK, client waits 2×MSL (~60–120s).
  Reason 1: If server didn't get the final ACK, it retransmits FIN →
            client can retransmit ACK.
  Reason 2: Prevents old delayed packets from polluting a new
            connection reusing the same port.
  Problem:  High-traffic servers may exhaust ephemeral ports.
  Solution: SO_REUSEADDR, or use QUIC (no TIME_WAIT needed).
```

---

### UDP — User Datagram Protocol

UDP is **connectionless, unreliable, and minimal** — adds only port multiplexing and an optional checksum on top of IP.

```
  ┌───────────────────────────────────┐
  │ Connectionless  (no handshake)    │
  │ Unreliable      (no retransmit)   │
  │ Unordered       (no seq numbers)  │
  │ Low overhead    (8-byte header)   │
  │ Low latency     (no ACK waiting)  │
  └───────────────────────────────────┘
```

#### UDP Datagram Structure

```
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |          Source Port          |       Destination Port        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |            Length             |           Checksum            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                             DATA                              |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

  Only 8 bytes of header — that's it.
  Checksum is optional in IPv4, mandatory in IPv6.
```

#### When to Use UDP

| Application        | Why UDP?                               |
|--------------------|----------------------------------------|
| DNS queries        | Single request/response, retry trivial |
| Video streaming    | Dropped frame better than delay        |
| VoIP / video calls | Real-time, latency critical            |
| Online gaming      | Position updates — stale = useless     |
| DHCP               | Broadcast before IP assigned           |
| QUIC (HTTP/3)      | Reliability reimplemented at app layer |

---

### TCP vs UDP — Full Comparison

```
  Feature               TCP                       UDP
  ─────────────────────────────────────────────────────────
  Connection            Required (3-way SYN)      None
  Reliability           Guaranteed delivery       Best effort
  Ordering              Strict (seq numbers)      None
  Retransmission        Yes (automatic)           No
  Error detection       Checksum (mandatory)      Checksum (opt v4)
  Flow control          Yes (window size)         No
  Congestion control    Yes (CWND)                No
  Header size           20–60 bytes               8 bytes (fixed)
  Latency               Higher (handshake cost)   Lower
  Broadcast/Multicast   No                        Yes
  Head-of-line blocking Yes                       No
  Use cases             HTTP/S, SSH, FTP, email   DNS, video, games
```

#### Latency Profile

```
  TCP request-response (e.g. HTTP):
  0ms  │── SYN ───────────────────────────────────────────────►│
  10ms │◄─ SYN-ACK ────────────────────────────────────────────│
  20ms │── ACK + HTTP GET ─────────────────────────────────────►│
  30ms │◄─ HTTP 200 response ───────────────────────────────────│
       First byte at ~30ms (3 RTTs minimum)

  UDP request-response (e.g. DNS):
  0ms  │── DNS query ──────────────────────────────────────────►│
  10ms │◄─ DNS response ────────────────────────────────────────│
       First byte at ~10ms (1 RTT)
```

#### Head-of-Line Blocking

```
  TCP — strict ordering causes blocking:
  [Pkt 1][Pkt 2 LOST][Pkt 3][Pkt 4]
  Receiver has 1, 3, 4 but MUST wait for 2 before delivering any.
  → Entire HTTP/2 stream stalls.

  UDP — no blocking:
  App receives whatever arrives, handles gaps itself.
  → Video player drops one frame, plays next. You barely notice.
```

#### Choosing TCP or UDP

```
  Does data NEED to arrive completely and in order?
         │                      │
        YES                     NO
         │                      │
        TCP             Is low latency critical?
                               │           │
                              YES          NO
                               │           │
                              UDP       Either works
```

---

### QUIC — The Modern Alternative (HTTP/3)

QUIC reimplements TCP's reliability over UDP with significant improvements.

```
  HTTP/1.1 & HTTP/2:          HTTP/3 (QUIC):
  ┌──────────────┐            ┌──────────────┐
  │   HTTP       │            │   HTTP/3     │
  ├──────────────┤            ├──────────────┤
  │   TLS        │            │   QUIC       │  ← reliability + TLS combined
  ├──────────────┤            ├──────────────┤
  │   TCP        │            │   UDP        │
  ├──────────────┤            ├──────────────┤
  │   IP         │            │   IP         │
  └──────────────┘            └──────────────┘

  QUIC advantages over TCP:
  - 0-RTT / 1-RTT setup     (TCP needs 1–3 RTTs)
  - No head-of-line blocking (each stream is independent)
  - Built-in TLS 1.3         (no separate handshake overhead)
  - Connection migration     (change IP without dropping session)
  - User-space implementation (faster iteration than kernel TCP)
```

---

### Transport Layer Quick Reference

| Term           | Meaning                                                          |
|----------------|------------------------------------------------------------------|
| Port           | 16-bit identifier for an application/service (0–65535)         |
| Socket         | IP + Port — one endpoint of a connection                        |
| 4-tuple        | (src IP, src port, dst IP, dst port) — uniquely IDs connection |
| TCP            | Reliable, ordered, connection-oriented — 3-way handshake        |
| UDP            | Fast, connectionless, no guarantees — 8-byte header             |
| SYN/ACK/FIN    | TCP control flags for setup and teardown                        |
| Sequence #     | Tracks byte position; enables ordering & retransmission         |
| ACK #          | Next expected byte from other side; triggers retransmit if gap  |
| Window Size    | Flow control — receiver tells sender how much buffer is free    |
| CWND           | Congestion window — limits in-flight data vs network capacity   |
| ISN            | Initial Sequence Number — random starting point (security)      |
| RTT            | Round Trip Time — time for packet to go and come back          |
| MSL            | Maximum Segment Lifetime — max age of packet in network         |
| TIME_WAIT      | Post-close state to absorb delayed duplicate packets            |
| Ephemeral port | Temp client port (49152–65535) assigned per outgoing connection |
| QUIC           | UDP-based modern transport with reliability built in (HTTP/3)   |

---

## 7. Layer 4 — Application Layer

> **OSI equivalent:** Session (L5) + Presentation (L6) + Application (L7)
> **Job:** Provide network services directly to user applications — the protocols that users and programs actually interact with.

```
  ┌─────────────────┐
  │ 4. Application  │  ◄── YOU ARE HERE
  ├─────────────────┤
  │ 3. Transport    │
  ├─────────────────┤
  │ 2. Internet     │
  ├─────────────────┤
  │ 1. Network      │
  │    Access       │
  └─────────────────┘
```

This layer includes what OSI separates into three layers:
- **Session (OSI L5):** Managing connection sessions (login, logout, checkpointing)
- **Presentation (OSI L6):** Data formatting, encryption (TLS), encoding (JSON, JPEG)
- **Application (OSI L7):** User-facing protocols (HTTP, DNS, FTP, SMTP, SSH)

### Common Application Layer Protocols

| Protocol | Port    | Transport | Purpose                         |
|----------|---------|-----------|---------------------------------|
| HTTP     | 80      | TCP       | Web (unencrypted)               |
| HTTPS    | 443     | TCP       | Web (encrypted via TLS)         |
| DNS      | 53      | UDP/TCP   | Domain name → IP lookup         |
| FTP      | 20/21   | TCP       | File transfer                   |
| SMTP     | 25      | TCP       | Sending email                   |
| POP3     | 110     | TCP       | Retrieving email (download)     |
| IMAP     | 143     | TCP       | Retrieving email (sync)         |
| SSH      | 22      | TCP       | Secure remote shell             |
| Telnet   | 23      | TCP       | Remote shell (insecure)         |
| DHCP     | 67/68   | UDP       | Auto IP address assignment      |
| SNMP     | 161     | UDP       | Network device monitoring       |
| NTP      | 123     | UDP       | Network time synchronization    |

### How This Layer Works

```
  User types: https://example.com

  Application layer action:
  ┌─────────────────────────────────────────────┐
  │ 1. DNS query (UDP/53)                       │
  │    "What is the IP of example.com?"         │
  │    Answer: "93.184.216.34"                  │
  ├─────────────────────────────────────────────┤
  │ 2. TLS handshake (Presentation sub-layer)   │
  │    Negotiate cipher suite, exchange certs   │
  │    Establish encrypted session              │
  ├─────────────────────────────────────────────┤
  │ 3. HTTP request (Application sub-layer)     │
  │    GET /index.html HTTP/1.1                 │
  │    Host: example.com                        │
  │    Accept: text/html                        │
  └─────────────────────────────────────────────┘
  All of this hands data DOWN to the Transport layer (TCP port 443).
```

### TLS — Encryption at the Presentation Sub-layer

TLS (Transport Layer Security) operates conceptually at the Presentation layer — it encrypts data before it passes to TCP.

```
  Plain HTTP:
  Browser ──── "GET /secret" ────────────────────────────────► Server
               (readable by anyone on the path)

  HTTPS (HTTP + TLS):
  Browser ──── "xK3$@!#mQ9..." (encrypted) ──────────────────► Server
               (only browser and server can decrypt)

  TLS Handshake (simplified):
  Client ──── ClientHello (cipher suites, TLS version) ──────►
  Client ◄─── ServerHello + Certificate ──────────────────────
  Client ──── Key exchange ────────────────────────────────────►
              (both sides derive same session key)
  Client ──── Encrypted data ─────────────────────────────────►
```

---

## 8. Data Encapsulation & the Full Journey

As data moves **down** the TCP/IP stack on the sender, each layer **wraps** the payload with its own header (encapsulation). At the receiver, each layer **strips** its own header as data moves **up** (decapsulation).

Each layer only reads and removes the header that belongs to it — it treats everything else as opaque payload.

---

### Part 1 — Encapsulation (Sender Side, Top → Bottom)

```
  TCP/IP Layer     What happens                        PDU name
  ─────────────────────────────────────────────────────────────────────

  4 Application  ┌────────────────────────────────┐
                 │  HTTP data                     │   Data
                 │  "GET /index.html HTTP/1.1"    │
                 └────────────────────────────────┘
                                │
                                │  Transport layer adds TCP header
                                ▼
  3 Transport    ┌────────────┬─┴─────────────────┐
                 │ TCP Header │  HTTP data         │   Segment
                 │ (src/dst   │                    │
                 │  port,seq, │                    │
                 │  ack,flags)│                    │
                 └────────────┴────────────────────┘
                                │
                                │  Internet layer adds IP header
                                ▼
  2 Internet     ┌──────────┬──┴────────────────────┐
                 │ IP Header│ TCP Header │ HTTP data  │   Packet
                 │(src/dst  │            │            │
                 │ IP, TTL, │            │            │
                 │ protocol)│            │            │
                 └──────────┴────────────┴────────────┘
                                │
                                │  Network Access adds MAC header + FCS trailer
                                ▼
  1 Network      ┌────────┬─────┴───────────────────┬─────┐
     Access      │MAC Hdr │IP Hdr│TCP Hdr│HTTP data  │ FCS │   Frame
                 │(dst MAC│      │       │           │     │
                 │ src MAC│      │       │           │     │
                 │ type)  │      │       │           │     │
                 └────────┴──────┴───────┴───────────┴─────┘
                                │
                                │  Converted to raw bits
                                ▼
  Physical       1 0 1 1 0 0 1 0 1 0 1 1 0 1 0 0 1 0 1 0 ...   Bits
                 ──────────────────────────────────────────────►
                        (electrical / optical / radio signal)
```

---

### Part 2 — Decapsulation (Receiver Side, Bottom → Top)

Each layer reads its own header, acts on it, then passes the payload **up**.

```
  Physical       Receives raw bits from the wire / wireless medium

                 1 0 1 1 0 0 1 0 1 0 1 1 0 1 0 0 1 0 1 0 ...
                                │
                                │  Reconstruct bits into a frame
                                ▼
  1 Network      ┌────────┬──────────────────────────────┬─────┐
     Access      │MAC Hdr │           payload            │ FCS │
                 └────┬───┴──────────────────────────────┴──┬──┘
                      │  Read dst MAC — is it for me? YES   │
                      │  Verify FCS   — error check OK      │
                      │  Strip MAC header + FCS             │
                      ▼
  2 Internet     ┌──────────┬────────────────────────────────┐
                 │ IP Header│           payload              │
                 └────┬─────┴────────────────────────────────┘
                      │  Read dst IP — is it for me? YES
                      │  Verify checksum
                      │  Read Protocol field → TCP (6)
                      │  Strip IP header
                      ▼
  3 Transport    ┌────────────┬──────────────────────────────┐
                 │ TCP Header │           payload            │
                 └─────┬──────┴──────────────────────────────┘
                        │  Read dst port → 443 (HTTPS)
                        │  Verify checksum
                        │  Reassemble segments in order (seq#)
                        │  Send ACK back to sender
                        │  Strip TCP header
                        ▼
  4 Application  ┌────────────────────────────────┐
                 │  HTTP data                     │
                 │  "GET /index.html HTTP/1.1"    │
                 └────────────────────────────────┘
                        │
                        ▼
                 Web server process handles the request
```

---

### Part 3 — The Complete End-to-End Picture

```
       SENDER (Your Browser)                    RECEIVER (Web Server)
  ┌──────────────────────────┐            ┌──────────────────────────┐
  │ 4. Application           │            │ 4. Application           │
  │   HTTP GET /index.html   │            │   Process HTTP request   │
  └────────────┬─────────────┘            └──────────────▲───────────┘
               │ + HTTP data                             │ read HTTP
  ┌────────────▼─────────────┐            ┌──────────────┴───────────┐
  │ 3. Transport             │            │ 3. Transport             │
  │   + TCP header           │            │   - TCP header           │
  │   (port 54321 → 443)     │            │   (port 443 ← 54321)     │
  └────────────┬─────────────┘            └──────────────▲───────────┘
               │ + TCP segment                           │ strip TCP
  ┌────────────▼─────────────┐            ┌──────────────┴───────────┐
  │ 2. Internet              │            │ 2. Internet              │
  │   + IP header            │            │   - IP header            │
  │   (192.168.1.5→93.x.x.x) │            │   (93.x.x.x←192.168.1.5) │
  └────────────┬─────────────┘            └──────────────▲───────────┘
               │ + IP packet                             │ strip IP
  ┌────────────▼─────────────┐            ┌──────────────┴───────────┐
  │ 1. Network Access        │            │ 1. Network Access        │
  │   + MAC header + FCS     │            │   - MAC header + FCS     │
  │   (AA:BB:CC → DD:EE:FF)  │────────────│   verify FCS, read MAC   │
  └──────────────────────────┘  wire/Wi-Fi└──────────────────────────┘
```

> **Key insight:** At intermediate **routers**, only Layers 1–2 (Network Access + Internet) are processed. The router strips the frame, reads the IP header to decide the next hop, then re-encapsulates in a **new frame** with updated MAC addresses. TCP/UDP headers and application data are **never touched by routers**.

---

### Part 4 — Nested Header Detail

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │  FRAME  (Network Access Layer)                                       │
  │                                                                      │
  │  ┌──────────┬──────────┬──────────┬──────────────────────┬───────┐  │
  │  │ Dst MAC  │ Src MAC  │EtherType │       PAYLOAD        │  FCS  │  │
  │  │ 6 bytes  │ 6 bytes  │ 2 bytes  │                      │4 bytes│  │
  │  └──────────┴──────────┴──────────┴──────────┬───────────┴───────┘  │
  │                                              │                      │
  │  ┌───────────────────────────────────────────┘                      │
  │  │  PACKET  (Internet Layer)                                        │
  │  │                                                                  │
  │  │  ┌────────┬───────┬──────┬────────────┬────────┬─────────────┐  │
  │  │  │Version │  TTL  │Proto │  Src IP    │ Dst IP │   PAYLOAD   │  │
  │  │  │ 4 bits │ 8 bits│8 bits│  4 bytes   │ 4 bytes│             │  │
  │  │  └────────┴───────┴──┬───┴────────────┴────────┴──────┬──────┘  │
  │  │                      │                                │         │
  │  │  ┌───────────────────┘                                │         │
  │  │  │  SEGMENT  (Transport Layer)                        │         │
  │  │  │                                                    │         │
  │  │  │  ┌──────────┬──────────┬──────────┬────────────┐  │         │
  │  │  │  │ Src Port │ Dst Port │  Seq #   │  APP DATA  │  │         │
  │  │  │  │ 2 bytes  │ 2 bytes  │  4 bytes │            │  │         │
  │  │  │  └──────────┴──────────┴──────────┴────────────┘  │         │
  │  │  │                    (Application Layer payload)     │         │
  │  └──┘                                                    │         │
  └──────────────────────────────────────────────────────────────────────┘

  FCS   = Frame Check Sequence — CRC error detection (Layer 1 trailer)
  Proto = Protocol field: 6=TCP, 17=UDP, 1=ICMP
```

---

### Example: Loading `https://example.com` End to End

```
  Step 1 — DNS  (Application → Transport/UDP → Internet → wire)
    Browser:  "What IP is example.com?"
    DNS:      "93.184.216.34"

  Step 2 — TCP 3-Way Handshake  (Transport layer)
    SYN ──────────────────────────────────────────────────────────►
    ◄────────────────────────────────────────────────── SYN-ACK
    ACK ──────────────────────────────────────────────────────────►

  Step 3 — TLS Handshake  (Application/Presentation layer, over TCP)
    Negotiate cipher, exchange certificates, derive session keys.

  Step 4 — HTTP Request built at Application layer
    GET /index.html HTTP/1.1
    Host: example.com

  Step 5 — Encapsulation going DOWN the stack:
    Application:   [HTTP data                                           ]
    Transport:     [TCP Hdr  | HTTP data                               ]
    Internet:      [IP Hdr   | TCP Hdr  | HTTP data                   ]
    Network Access:[MAC Hdr  | IP Hdr   | TCP Hdr  | HTTP data | FCS  ]
    Physical:      1 0 1 1 0 0 1 0 1 0 ...  ─────────────────────────►

  Step 6 — Decapsulation going UP at the server (exact reverse)

  Step 7 — Server sends HTTP 200 OK response (same process in reverse)
```

---

## 9. NAT — Network Address Translation

NAT operates at the boundary between the **Internet layer** and the **Network Access layer** — typically implemented in a router.

### The Concept

Think of NAT like a **corporate mailroom**. The building has one street address (the **public IP**), but hundreds of employees have their own desk numbers (**private IPs**). When an employee sends a letter out, the mailroom replaces their desk number with the building's address so the reply comes back to the right place.

### How PAT (Port Address Translation) Works

PAT is the most common variant — multiple devices share **one public IP** using unique port numbers to track each session.

```
     PRIVATE NETWORK (Local)              |          PUBLIC INTERNET
                                          |
 [Laptop] 192.168.1.10:5001  ───────┐    |
 [Phone]  192.168.1.11:5002  ───────┼──[ROUTER]──── Public IP: 203.0.113.1
 [Tablet] 192.168.1.12:5003  ───────┘    |
                                          |
──────────────────────────────────────────────────────────────────────
          NAT TRANSLATION TABLE           |       OUTSIDE WORLD
──────────────────────────────────────────
  Internal IP:Port   ◄──►  External Port  |   (Website sees everyone
  192.168.1.10:5001  ◄──►  Port 10001    |    as 203.0.113.1)
  192.168.1.11:5002  ◄──►  Port 10002    |          [Web Server]
  192.168.1.12:5003  ◄──►  Port 10003    |         IP: 8.8.8.8
──────────────────────────────────────────────────────────────────────
```

### The Flow

```
  1. Outbound:
     Laptop (192.168.1.10:5001) sends packet to 8.8.8.8:80
                    │
                    ▼
     Router rewrites: Src IP:Port = 203.0.113.1:10001
     Router records: 10001 → 192.168.1.10:5001  (in NAT table)
                    │
                    ▼
     Packet reaches 8.8.8.8 — looks like it came from 203.0.113.1

  2. Inbound:
     8.8.8.8 replies to 203.0.113.1:10001
                    │
                    ▼
     Router looks up 10001 in NAT table → 192.168.1.10:5001
     Router rewrites: Dst IP:Port = 192.168.1.10:5001
                    │
                    ▼
     Packet delivered to Laptop
```

### Types of NAT

| Type               | Description                                               | Use Case                          |
|--------------------|-----------------------------------------------------------|-----------------------------------|
| **Static NAT**     | Fixed 1-to-1 mapping: one private IP ↔ one public IP     | Hosting a server reachable from internet |
| **Dynamic NAT**    | Maps private IPs to a pool of public IPs (first-come)    | Large enterprise outbound traffic |
| **PAT (Overload)** | Many private IPs → one public IP, differentiated by port | Home routers, small offices       |

### Why NAT Matters

- **IP Conservation:** Thousands of devices share a single public IPv4 address, delaying IPv4 exhaustion.
- **Security:** External hosts cannot initiate connections to private devices (no inbound mapping exists until an outbound connection creates one).
- **Flexibility:** Change internal IP scheme or ISP without reconfiguring every device.

> **Note:** IPv6's 340 undecillion addresses eliminate the *need* for NAT — every device can have a globally unique routable address. IPv6 was designed with end-to-end connectivity in mind.

---

## 10. Key Concepts at a Glance

| Concept          | Layer         | Description                                                   |
|------------------|---------------|---------------------------------------------------------------|
| MAC Address      | Network Access| 48-bit hardware address; used for local frame delivery        |
| ARP              | Network Access| Resolves IP → MAC on the local network                       |
| Ethernet Frame   | Network Access| PDU at Layer 1; contains MAC header + payload + FCS          |
| IP Address       | Internet      | 32-bit (v4) or 128-bit (v6) logical address for routing      |
| CIDR             | Internet      | /prefix notation for subnet masks (e.g. /24 = 254 hosts)     |
| Private IPs      | Internet      | 10.x, 172.16–31.x, 192.168.x — not routable on internet      |
| Loopback         | Internet      | 127.0.0.1 (IPv4), ::1 (IPv6) — refers to local machine       |
| TTL / Hop Limit  | Internet      | Decremented per router hop; 0 = drop + ICMP Time Exceeded    |
| ICMP             | Internet      | Error & diagnostic protocol — no data transport              |
| ping             | Internet      | ICMP Type 8/0 — tests reachability, measures RTT             |
| traceroute       | Internet      | Maps path using TTL + ICMP Time Exceeded messages            |
| Port             | Transport     | 16-bit number identifying an application/service             |
| Socket           | Transport     | IP + Port — one endpoint of a connection                     |
| 4-tuple          | Transport     | (src IP, src port, dst IP, dst port) — uniquely IDs a conn   |
| TCP              | Transport     | Reliable, ordered, connection-oriented; 3-way handshake      |
| UDP              | Transport     | Fast, connectionless, no guarantees; 8-byte header           |
| 3-Way Handshake  | Transport     | TCP setup: SYN → SYN-ACK → ACK                              |
| Flow Control     | Transport     | Receiver controls send rate via Window Size field            |
| Congestion Ctrl  | Transport     | TCP adjusts CWND based on inferred network congestion        |
| QUIC             | Transport     | UDP-based protocol with reliability built in (HTTP/3)        |
| HTTP/HTTPS       | Application   | Web protocol; HTTPS adds TLS encryption                      |
| DNS              | Application   | Resolves domain names to IP addresses (UDP/53)               |
| TLS              | Application   | Encryption layer for HTTPS, SMTP, etc.                       |
| NAT/PAT          | Internet edge | Translates private IPs to public IP using port mapping       |
| MTU              | Network Access| Maximum packet size on a link (typically 1500 bytes)         |
