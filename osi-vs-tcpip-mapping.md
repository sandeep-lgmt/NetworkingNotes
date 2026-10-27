# How OSI Maps to the Real-World TCP/IP Model

> Beginner-friendly notes. Read [The OSI Model](osi-model-basics.md) first; this note builds directly on it. Earlier notes: [Networking Basics](networking-basics.md), [IPv4](ipv4-basics.md), [Subnetting](subnetting-basics.md), [ARP and MAC](arp-and-mac-basics.md), [IPv6](ipv6-basics.md).

## Table of Contents

1. [Two Models, One Network](#1-two-models-one-network)
2. [The TCP/IP Model (4 Layers)](#2-the-tcpip-model-4-layers)
3. [Side-by-Side Mapping](#3-side-by-side-mapping)
4. [Why OSI Layers 5-6-7 Collapse Into One](#4-why-osi-layers-5-6-7-collapse-into-one)
5. [Why OSI Layers 1-2 Collapse Into One](#5-why-osi-layers-1-2-collapse-into-one)
6. [Where Real Protocols Live](#6-where-real-protocols-live)
7. [The Gray Areas](#7-the-gray-areas)
8. [How Each Layer Knows What to Hand Up](#8-how-each-layer-knows-what-to-hand-up)
9. [Full Walkthrough: Opening a Website](#9-full-walkthrough-opening-a-website)
10. [Key Differences Between the Models](#10-key-differences-between-the-models)
11. [Which Model Do I Use When?](#11-which-model-do-i-use-when)
12. [Hands-On Lab with Wireshark](#12-hands-on-lab-with-wireshark)
13. [Practice Questions](#13-practice-questions)
14. [Quick Cheat Sheet](#14-quick-cheat-sheet)

---

## 1. Two Models, One Network

You have learned the 7-layer **OSI model**. But if you open any real device, operating system, or Internet router, you will not find "a Session layer" or "a Presentation layer" as separate pieces. What is actually running is the **TCP/IP protocol suite**.

So why do both exist?

| | OSI model | TCP/IP model |
|---|-----------|--------------|
| **Origin** | Created by **ISO** (standardized in 1984) as a vendor-neutral blueprint | Grew out of the **US research network (ARPANET)** in the 1970s; ARPANET switched to TCP/IP in **1983** |
| **How it was made** | **Model first, protocols later.** Committees designed the layers, then tried to write protocols for them | **Protocols first, model later.** Working protocols (TCP, IP) were built and used; the model describes what they do |
| **What happened** | The OSI protocols were complex and late, and never won widespread use | Simple, working, and freely available, so it spread worldwide |
| **Today** | A **teaching and troubleshooting vocabulary** | The **actual implementation** of the Internet |

**Analogy:** OSI is like a detailed architectural blueprint that was never fully built. TCP/IP is the house people actually moved into; its layout was described afterwards.

> The most useful thing to remember: **OSI = the language we use to talk about networks. TCP/IP = what is actually running.**

---

## 2. The TCP/IP Model (4 Layers)

The classic TCP/IP model (described in **RFC 1122**) has **4 layers**:

| # | TCP/IP layer | Job | Real protocols |
|---|--------------|-----|----------------|
| 4 | **Application** | Everything the application needs: protocols, data format, encryption, session handling | HTTP, HTTPS, DNS, DHCP, SMTP, SSH, FTP, TLS |
| 3 | **Transport** | Delivery between programs (ports), reliability | TCP, UDP, QUIC |
| 2 | **Internet** | Logical addressing and routing across networks | IPv4, IPv6, ICMP, IPsec |
| 1 | **Link** (also "Network Access" or "Network Interface") | Delivery on the local network, plus the physical signals | Ethernet, Wi-Fi, ARP, PPP |

Note the naming trap: the TCP/IP **"Internet" layer** is OSI's **"Network" layer (3)**, and the TCP/IP **"Link" layer** is where OSI's layers 1 and 2 live.

### The popular 5-layer hybrid

Many textbooks and courses use a **5-layer** model, which splits the bottom layer back into two. This is the most practical teaching version:

| # | Hybrid 5-layer |
|---|----------------|
| 5 | Application |
| 4 | Transport |
| 3 | Network |
| 2 | Data Link |
| 1 | Physical |

So you will meet **7-layer (OSI)**, **5-layer (hybrid)**, and **4-layer (TCP/IP)** versions. They describe the same real network; they just group it differently.

---

## 3. Side-by-Side Mapping

```
   OSI (7 layers)          Hybrid (5 layers)       TCP/IP (4 layers)
 +-------------------+   +-------------------+   +-------------------+
 | 7  Application    |   |                   |   |                   |
 +-------------------+   |                   |   |                   |
 | 6  Presentation   |   | 5  Application    |   | 4  Application    |
 +-------------------+   |                   |   |                   |
 | 5  Session        |   |                   |   |                   |
 +-------------------+   +-------------------+   +-------------------+
 | 4  Transport      |   | 4  Transport      |   | 3  Transport      |
 +-------------------+   +-------------------+   +-------------------+
 | 3  Network        |   | 3  Network        |   | 2  Internet       |
 +-------------------+   +-------------------+   +-------------------+
 | 2  Data Link      |   | 2  Data Link      |   |                   |
 +-------------------+   +-------------------+   | 1  Link           |
 | 1  Physical       |   | 1  Physical       |   |  (Network Access) |
 +-------------------+   +-------------------+   +-------------------+
```

### The same mapping as a table

| OSI layer | TCP/IP layer | Notes |
|-----------|--------------|-------|
| 7 Application | **Application** | One-to-one at the top |
| 6 Presentation | **Application** | Merged in |
| 5 Session | **Application** | Merged in |
| 4 Transport | **Transport** | Clean one-to-one match |
| 3 Network | **Internet** | Clean one-to-one match (different name) |
| 2 Data Link | **Link** | Merged |
| 1 Physical | **Link** | Merged |

**Layers 4 and 3 line up perfectly.** That is why "Layer 4" always means TCP/UDP/ports and "Layer 3" always means IP/routing, no matter which model you use. The differences are all at the **top** (3 layers into 1) and the **bottom** (2 layers into 1).

---

## 4. Why OSI Layers 5-6-7 Collapse Into One

In the OSI model, the top three layers have separate jobs. In real life, **the application itself does all three jobs**, so TCP/IP does not bother separating them.

| OSI layer | OSI job | In practice |
|-----------|---------|-------------|
| 7 Application | Protocol for the app | The application and its protocol (HTTP, DNS...) |
| 6 Presentation | Encode, encrypt, compress | Done by the application or a **library** (a TLS library, image/video codecs, `gzip`) |
| 5 Session | Open, manage, close dialogues | Done by the application protocol or the app's code (cookies, tokens, login sessions, connection handling) |

**Why this works out in practice:**

- There is no standard "Presentation protocol" that every program plugs into. A browser handles encoding and compression itself, and calls a TLS library for encryption.
- Session management usually is part of the application (a website login cookie) or part of the protocol itself.
- Putting it all in one layer gives programmers **flexibility**: each application decides how to format, secure, and manage its data.

**Example:** when you open an HTTPS site, the "Presentation" work (encryption with TLS, compression with gzip, UTF-8 text encoding) and the "Session" work (cookies, login) are all done *by or for the browser and web server*, with no separate layers in between.

---

## 5. Why OSI Layers 1-2 Collapse Into One

At the bottom, the reason is different. The standards that define LAN technologies **cover both layers in one document**:

| Standard | What it defines |
|----------|-----------------|
| **IEEE 802.3 (Ethernet)** | The physical signals/cables **and** the frame format with MAC addressing |
| **IEEE 802.11 (Wi-Fi)** | The radio transmission **and** the frames, addressing, and access rules |

You cannot really mix and match: an Ethernet frame is designed to be sent over Ethernet cabling. Since both are one package, TCP/IP treats them as a single **Link** layer, and TCP/IP only cares about one thing: *"get this packet to the next device on this link."*

The 5-layer hybrid model separates them again because it is easier to teach (cables and signals are very different from frames and MAC addresses).

---

## 6. Where Real Protocols Live

| Protocol | What it does | OSI layer | TCP/IP layer |
|----------|--------------|-----------|--------------|
| **HTTP** | Web requests | 7 | Application |
| **HTTPS** (HTTP over **TLS**) | Secure web | 7 (TLS: 6, in practice between 4 and 7) | Application |
| **DNS** | Names to IPs | 7 | Application |
| **DHCP** | Hands out IP configuration | 7 | Application |
| **SMTP / IMAP / POP3** | Email | 7 | Application |
| **SSH** | Secure remote login | 7 | Application |
| **FTP** | File transfer | 7 | Application |
| **TCP** | Reliable delivery, ports | 4 | Transport |
| **UDP** | Fast, no-guarantee delivery, ports | 4 | Transport |
| **QUIC** | Reliable + encrypted transport on top of UDP (used by HTTP/3) | 4 | Transport |
| **IPv4 / IPv6** | Addressing and routing | 3 | Internet |
| **ICMP** (ping, traceroute) | Errors and diagnostics | 3 | Internet |
| **IPsec** | Secure IP | 3 | Internet |
| **ARP** | IP to MAC | 2 to 3 (see gray areas) | Link |
| **NDP** (IPv6, built on ICMPv6) | IPv6 neighbor discovery | 3 | Internet |
| **Ethernet (802.3)** | Wired LAN | 2 + 1 | Link |
| **Wi-Fi (802.11)** | Wireless LAN | 2 + 1 | Link |
| **VLAN (802.1Q)** | Virtual LANs | 2 | Link |
| **Cables, fiber, radio** | Carry the signal | 1 | Link |

---

## 7. The Gray Areas

Real protocols do not always sit neatly in one layer. These are the common "it depends" cases, and good ones to know for interviews.

| Protocol | The puzzle | The usual answer |
|----------|-----------|------------------|
| **ARP** | It uses MAC addresses (L2) but resolves IP addresses (L3) | Usually placed at **Layer 2** (it never leaves the local link), sometimes described as "Layer 2.5". In TCP/IP it belongs to the **Link** layer. |
| **ICMP** | It is carried inside IP packets, like TCP and UDP, but it is part of making IP work | **Layer 3** / Internet layer. It has **no ports**, so it is not a transport protocol. |
| **TLS** | Encrypts data (feels like L6) but runs on top of TCP and below HTTP | Described at **L6**, in practice sits **between 4 and 7**. In TCP/IP it is just part of the Application layer. |
| **DHCP** | It configures the *network* layer, but it is an application running over UDP (ports 67/68) | **Layer 7** (Application). |
| **BGP** | It does routing (a network-layer job), but runs over **TCP port 179** | Technically **Layer 7**, but functionally a routing protocol. |
| **OSPF** | A routing protocol that runs **directly over IP** (IP protocol number 89), with no TCP/UDP | **Layer 3** (Internet layer). |
| **VPNs** | Tunnels carry packets inside packets, and can be at L2, L3, or L7 depending on the type | Depends on the VPN technology. |
| **MAC address** | Part of the Link layer, but also used as an identifier in ARP and DHCP | **Layer 2**. |

**The lesson:** layers are a *guide* to how responsibilities are divided, not walls. When someone asks "what layer is X?", it is fine to answer: *"officially Y, but in practice it sits between Z and W because..."*

---

## 8. How Each Layer Knows What to Hand Up

When a frame arrives, how does the receiving device know **which protocol** should get the data inside? Every layer's header contains a **"next protocol" field**, which is the glue that makes layering work:

| Layer | Field in the header | Tells the receiver | Examples |
|-------|---------------------|--------------------|----------|
| **Link (Ethernet)** | **EtherType** | Which Layer 3 protocol is inside | `0x0800` = IPv4, `0x86DD` = IPv6, `0x0806` = ARP |
| **Internet (IP)** | **Protocol** field (IPv4) / Next Header (IPv6) | Which Layer 4 protocol is inside | `6` = TCP, `17` = UDP, `1` = ICMP, `89` = OSPF |
| **Transport (TCP/UDP)** | **Destination port** | Which application gets the data | `443` = HTTPS, `53` = DNS, `22` = SSH |

```
Frame arrives
  +-- EtherType = 0x0800   -> hand the payload to IPv4
        +-- Protocol = 6   -> hand the payload to TCP
              +-- Port = 443  -> hand the data to the web server (HTTPS)
```

This chain is called **demultiplexing**, and is a good way to remember how a single network card can serve many programs at once.

---

## 9. Full Walkthrough: Opening a Website

Everything from this series, in the order it actually happens when you type `https://www.example.com` in a browser. PC A: `192.168.1.10`, gateway `192.168.1.1`.

| Step | What happens | Protocol | OSI layer | TCP/IP layer |
|------|--------------|----------|-----------|--------------|
| 1 | The PC already got its IP, mask, gateway, and DNS server when it joined the network | **DHCP** (UDP 67/68) | 7 | Application |
| 2 | Browser asks "what is the IP of `www.example.com`?" | **DNS** (UDP 53) | 7 | Application |
| 3 | The DNS server is not on the local network, so the PC needs the **gateway's MAC** | **ARP** (broadcast, then reply) | 2 | Link |
| 4 | DNS request goes out in an IP packet, in a frame to the gateway. The reply returns `203.0.113.50` | UDP + IP + Ethernet | 4 / 3 / 2 | Transport / Internet / Link |
| 5 | The PC opens a connection to `203.0.113.50:443` | **TCP** three-way handshake | 4 | Transport |
| 6 | The two sides agree on encryption keys | **TLS** handshake | 6 (in practice 4-7) | Application |
| 7 | Browser sends the encrypted HTTP request | **HTTP** `GET /` | 7 | Application |
| 8 | The request is cut into segments, put into IP packets, and put into frames (new MACs at every router hop) | TCP, IP, Ethernet | 4 / 3 / 2 | Transport / Internet / Link |
| 9 | Frames become signals on the cable or radio | Ethernet or Wi-Fi physical layer | 1 | Link |
| 10 | The server replies with `200 OK` and the HTML; the browser decrypts and displays the page | HTTP over TLS over TCP | 7 | Application |

### The same request seen through TCP/IP's 4 layers (going down)

```
APPLICATION    HTTP request, TLS encryption, DNS lookup
   |           "GET /index.html"                            data
   v
TRANSPORT      TCP header: ports 51514 -> 443                segment
   |
   v
INTERNET       IP header: 192.168.1.10 -> 203.0.113.50       packet
   |
   v
LINK           Ethernet header/trailer: MAC AA..01 -> CC..03 frame  --> bits
```

The **application** layer does the work of OSI layers 7+6+5 together, while the **link** layer does OSI 2+1 together.

---

## 10. Key Differences Between the Models

| Feature | OSI | TCP/IP |
|---------|-----|--------|
| Type | **Reference model** (a theoretical blueprint) | **Implementation model** (describes real protocols) |
| Layers | 7 | 4 (or 5 in the hybrid) |
| Developed by | ISO | US Department of Defense research (ARPANET) |
| Approach | Model first, protocols later | Protocols first, model later |
| Session and Presentation | Separate layers | Part of the Application layer |
| Physical and Data Link | Separate layers | Combined into Link (or separate in the hybrid) |
| Strictness | **Strict**: each layer has a clear, separate job | **Flexible**: layers overlap, protocols cross boundaries |
| Services / interfaces / protocols | Clearly separated concepts | Less clearly separated |
| Used for | Teaching, design discussion, troubleshooting | Real Internet communication |
| Classic textbook contrast* | Network layer supports both connectionless and connection-oriented service; the transport layer is connection-oriented | The Internet layer is connectionless (IP); the transport layer offers both (TCP and UDP) |

\* This last row is the traditional textbook comparison. Do not worry if it feels abstract: the practical takeaway is **IP is connectionless, and TCP adds reliability on top of it.**

---

## 11. Which Model Do I Use When?

| Situation | Use | Why |
|-----------|-----|-----|
| Learning how networks work | **OSI** (or the 5-layer hybrid) | Finer detail makes each job clear |
| Troubleshooting | **OSI numbers** | "Check Layer 1, then Layer 2..." is a standard method |
| Talking to other engineers | **OSI numbers** | "That's a **Layer 2** switch", "a **Layer 7** firewall", "an **L4** load balancer" |
| Describing real protocols or writing network code | **TCP/IP** | It matches how software is actually built |
| Certifications (CCNA, Network+) | **Both**: you must know OSI and TCP/IP and how they map | Frequent exam topic |

### The industry quirk worth remembering

Even when engineers talk about the real TCP/IP network, they use **OSI layer numbers**:

| Phrase | Means |
|--------|-------|
| "Layer 2 switch" | Forwards by MAC address |
| "Layer 3 switch / router" | Forwards by IP address |
| "Layer 4 load balancer" | Balances by IP + port (TCP/UDP) |
| "Layer 7 load balancer / firewall" | Looks inside HTTP and other application data |

So you will use **both** in your career: TCP/IP for what actually runs and OSI for naming and troubleshooting.

---

## 12. Hands-On Lab with Wireshark

Wireshark (free) shows the layers in real packets, and it displays them in **TCP/IP order, bottom to top**.

1. **Install Wireshark** and start a capture on your active network interface (Wi-Fi or Ethernet).
2. In your browser, open a plain **HTTP** site (many sites redirect to HTTPS; a local router admin page is a safe option), or use `ping` for a simpler test.
3. **Stop the capture.** Type `http` (or `icmp`) in the display filter bar and press Enter.
4. **Click one packet.** In the middle pane you will see layered headers, for example:

   ```
   Frame 52: ...                       <- Layer 1 (the captured bits, info about the capture)
   Ethernet II, Src: ..., Dst: ...     <- Layer 2   (Link layer: MAC addresses)
   Internet Protocol Version 4, ...    <- Layer 3   (Internet layer: IP addresses)
   Transmission Control Protocol, ...  <- Layer 4   (Transport layer: ports)
   Hypertext Transfer Protocol         <- Layer 7   (Application layer)
   ```

5. **Expand each line** and find these:
   - Link: source/destination **MAC** and the **EtherType** (`0x0800`).
   - Internet: source/destination **IP**, **TTL**, **Protocol** (`6` for TCP).
   - Transport: source/destination **ports**, **flags** (SYN, ACK...).
   - Application: the actual `GET` request or response.
6. **Capture a three-way handshake:** filter `tcp.flags.syn == 1` and look for SYN, SYN-ACK, then ACK.
7. **Capture ARP:** filter `arp`, clear your ARP cache (see the [ARP notes](arp-and-mac-basics.md)), then ping a LAN device.
8. **Capture DNS:** filter `dns` and open a new website; look for UDP port 53.

Only capture on networks you own or have permission to monitor.

**What you will notice:** the Wireshark pane is literally encapsulation made visible: one header per layer, stacked, with the "next protocol" fields from section 8 pointing from one layer to the next.

---

## 13. Practice Questions

1. How many layers does the TCP/IP model have? Name them.
2. Which TCP/IP layer is equivalent to OSI Layer 3?
3. Which OSI layers are combined into TCP/IP's Application layer?
4. Which OSI layers are combined into TCP/IP's Link layer?
5. Which OSI layers match TCP/IP **exactly**?
6. Why did the TCP/IP model win over OSI?
7. Why does the TCP/IP model not have separate Session and Presentation layers?
8. Place these in the TCP/IP model: DNS, TCP, IP, Ethernet, ICMP, HTTP, ARP, UDP.
9. What is the job of the EtherType field? The IP Protocol field? The destination port?
10. Why can't ICMP be a Transport-layer protocol, even though it rides inside IP just like TCP and UDP?
11. An engineer says "we need a Layer 7 firewall". What does the "7" refer to, and is it from OSI or TCP/IP?
12. Why does Wireshark show Ethernet first and HTTP last?

<details>
<summary>Show answers</summary>

1. **Four**: Application, Transport, Internet, Link (Network Access). The popular hybrid version has five (it splits Link into Data Link and Physical).
2. The **Internet** layer.
3. OSI layers **7, 6, and 5** (Application, Presentation, Session).
4. OSI layers **2 and 1** (Data Link and Physical).
5. **Transport (4)** and **Network (3)**, which is the **Internet** layer in TCP/IP. They line up one-to-one.
6. It was **working, simple, and freely available early on**, and was already in use on ARPANET. OSI protocols were complex and arrived late.
7. In real software, **the application (or a library) already handles** formatting, encryption, and session tracking, so one layer is enough.
8. **Application:** DNS, HTTP. **Transport:** TCP, UDP. **Internet:** IP, ICMP. **Link:** Ethernet, ARP (ARP is usually placed at the Link layer).
9. **EtherType:** tells which Layer 3 protocol is inside the frame (IPv4, IPv6, ARP). **IP Protocol field:** tells which Layer 4 protocol is inside (TCP = 6, UDP = 17, ICMP = 1). **Destination port:** tells which application should get the data.
10. ICMP **has no ports** and is not used to carry application data between programs; it supports IP itself (errors, ping). So it is classified at the Internet/Network layer.
11. The "7" is the **OSI layer number** (Application). Engineers use OSI numbering even when describing the TCP/IP-based real network.
12. Wireshark lists headers in the order they appear in the packet: the outermost (Link layer, the Ethernet header) comes first, and the innermost data (Application) last.

</details>

---

## 14. Quick Cheat Sheet

| OSI | Hybrid (5) | TCP/IP (4) | Key examples |
|-----|------------|------------|--------------|
| 7 Application | 5 Application | **Application** | HTTP, DNS, DHCP, SMTP, SSH |
| 6 Presentation | | | TLS, encoding, compression |
| 5 Session | | | session handling, cookies |
| 4 Transport | 4 Transport | **Transport** | TCP, UDP, QUIC, ports |
| 3 Network | 3 Network | **Internet** | IP, ICMP, IPsec |
| 2 Data Link | 2 Data Link | **Link** | Ethernet, Wi-Fi, ARP, MAC |
| 1 Physical | 1 Physical | | Cables, radio, fiber |

| Remember | |
|----------|---|
| Layers 3 and 4 | Match **exactly** in both models |
| Top 3 OSI layers | Become **one** Application layer |
| Bottom 2 OSI layers | Become **one** Link layer |
| "Next protocol" fields | EtherType, IP Protocol, and Port glue the layers together |
| OSI | The **vocabulary**: teaching and troubleshooting |
| TCP/IP | The **reality**: what actually runs the Internet |
| Gray areas | ARP, ICMP, TLS, DHCP, BGP, OSPF |

### Remember in one sentence

> **TCP/IP is what really runs the Internet, and OSI is the more detailed language we use to describe it: they match exactly at the Transport and Network layers, TCP/IP merges OSI's top three layers into "Application" and its bottom two into "Link".**

---

*Notes by [your name]. Feel free to open an issue or PR if you spot a mistake.*
