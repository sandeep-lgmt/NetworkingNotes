# The OSI Model: All 7 Layers, from HTTP (L7) Down to Physical (L1)

> Beginner-friendly notes. This ties together everything in the series: [Networking Basics](networking-basics.md), [IPv4](ipv4-basics.md), [Subnetting](subnetting-basics.md), [ARP and MAC](arp-and-mac-basics.md), [IPv6](ipv6-basics.md).

## Table of Contents

1. [What Is the OSI Model?](#1-what-is-the-osi-model)
2. [The Big Picture](#2-the-big-picture)
3. [Layer 7: Application](#3-layer-7-application)
4. [Layer 6: Presentation](#4-layer-6-presentation)
5. [Layer 5: Session](#5-layer-5-session)
6. [Layer 4: Transport](#6-layer-4-transport)
7. [Layer 3: Network](#7-layer-3-network)
8. [Layer 2: Data Link](#8-layer-2-data-link)
9. [Layer 1: Physical](#9-layer-1-physical)
10. [Encapsulation: One Web Request Through All Layers](#10-encapsulation-one-web-request-through-all-layers)
11. [OSI vs TCP/IP](#11-osi-vs-tcpip)
12. [Troubleshooting by Layer](#12-troubleshooting-by-layer)
13. [Practice Questions](#13-practice-questions)
14. [Quick Cheat Sheet](#14-quick-cheat-sheet)

---

## 1. What Is the OSI Model?

**OSI = Open Systems Interconnection.** It is a **reference model** that splits all the work of networking into **7 layers**. Each layer has **one job** and only talks to the layers directly above and below it.

### Why split it into layers?

- **Easier to learn:** one small job at a time.
- **Easier to troubleshoot:** "Is it the cable (L1)? The IP (L3)? The app (L7)?"
- **Mix and match:** you can swap Wi-Fi for Ethernet (L1/L2) without changing HTTP (L7).
- **Common language:** "that's a Layer 2 problem" means the same thing to every engineer.

**Analogy:** sending a letter. You write it (content), put it in an envelope (addressing), the post office sorts it (routing), and trucks carry it (transport). Each step does one job and does not care how the others work.

### Important honesty note

OSI is a **teaching and reference model**. The real Internet runs on **TCP/IP**, which has fewer layers (section 11). So the boundaries between OSI layers 5, 6, and 7 are blurry in real life. But the 7 layers are still the standard way to describe and troubleshoot networks.

### Mnemonics (to remember the order)

| Direction | Mnemonic | Letters |
|-----------|----------|---------|
| **Top to bottom** (7 to 1) | **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing | Application, Presentation, Session, Transport, Network, Data Link, Physical |
| **Bottom to top** (1 to 7) | **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way | Physical, Data Link, Network, Transport, Session, Presentation, Application |

---

## 2. The Big Picture

```
 Layer   Name           Data unit (PDU)    What it handles               Example
 -----   ------------   ----------------   ---------------------------   ----------------------
   7     Application    Data               Network services for apps     HTTP, DNS, SMTP
   6     Presentation   Data               Format, encrypt, compress     TLS, JPEG, UTF-8
   5     Session        Data               Start/manage/end dialogues    Session setup, RPC
   -----------------------------------------------------------------------------------------
   4     Transport      Segment/Datagram   Process-to-process delivery   TCP, UDP (ports)
   3     Network        Packet             Routing between networks      IP, ICMP (IP addresses)
   2     Data Link      Frame              Delivery on the local link    Ethernet, Wi-Fi (MACs)
   1     Physical       Bits               Signals over the medium       Cables, radio, light
```

- **Layers 7, 6, 5**: "software" layers, about the **application** and its data.
- **Layers 4, 3, 2, 1**: the "data flow" layers, about **moving the data**.
- **PDU** = Protocol Data Unit = what the data is called at that layer.

### Which address or identifier belongs to which layer

| Layer | Identifier | Example |
|-------|-----------|---------|
| 7 | Name / URL | `www.example.com` |
| 4 | Port number | `443` |
| 3 | IP address | `203.0.113.50` |
| 2 | MAC address | `CC:CC:CC:CC:CC:03` |

We will follow **one web request** down through all of them. The example client is PC A from the ARP notes: IP `192.168.1.10`, MAC `AA:AA:AA:AA:AA:01`, gateway MAC `CC:CC:CC:CC:CC:03`.

> We start at the top (where the user is) and go down (where the wire is).

---

## 3. Layer 7: Application

### Job

Gives **network services to applications**. This is the layer where **protocols that applications use** live: "how do we ask for a web page?", "how do we send an email?"

> **Layer 7 is not the application itself.** Chrome is the *program*. **HTTP** is the Layer 7 *protocol* that Chrome uses.

### HTTP in detail (our starting point)

**HTTP = HyperText Transfer Protocol.** It is a request/response protocol: the client asks, the server answers.

A request:

```
GET /index.html HTTP/1.1
Host: www.example.com
```

A response:

```
HTTP/1.1 200 OK
Content-Type: text/html

<html> ... </html>
```

| HTTP concept | Meaning |
|--------------|---------|
| **Methods** | `GET` (fetch), `POST` (send data), `PUT`, `DELETE`... |
| **Status codes** | `200` OK, `301` moved, `404` not found, `500` server error |
| **Stateless** | Each request is independent; the server does not remember you unless cookies or tokens are used |
| **HTTPS** | HTTP carried inside an encrypted **TLS** connection |

### Other Layer 7 protocols

| Protocol | Purpose | Default port |
|----------|---------|--------------|
| HTTP | Web pages | 80 |
| HTTPS | Secure web | 443 |
| DNS | Names to IP addresses | 53 |
| DHCP | Gives out IP addresses | 67/68 (UDP) |
| SMTP | Sends email | 25 (587 for submission) |
| IMAP / POP3 | Reads email | 143 / 110 |
| FTP | File transfer | 21 |
| SSH | Secure remote login | 22 |

### What happens here in our example

The browser creates the request `GET /index.html`. Before that, it used **DNS** (also Layer 7) to turn `www.example.com` into an IP address, say `203.0.113.50`.

**Analogy:** writing the letter.

| | |
|---|---|
| Data unit | Data |
| Devices | Firewalls (application-aware), proxies, load balancers (L7) |

---

## 4. Layer 6: Presentation

### Job

Makes sure the data is in a **form both sides understand**, and handles **encryption** and **compression**. Think "translator and packager".

| Function | Examples |
|----------|----------|
| **Translation / encoding** | ASCII, UTF-8 (text), JPEG, PNG, GIF (images), MP3, MP4 |
| **Encryption / decryption** | TLS/SSL (HTTPS) |
| **Compression** | gzip, image and video compression |

### What happens here in our example

For HTTPS, the HTTP request is **encrypted** with TLS so nobody on the way can read it. Text is encoded as bytes (UTF-8).

### Honest note

In real networks there is no separate "Presentation protocol". These functions happen inside the application or in libraries. **TLS** is usually *described* at layer 6 but in practice sits between layers 4 and 7. That is fine: the model is for understanding.

**Analogy:** translating the letter into a language the receiver reads, and sealing it in a locked envelope.

| | |
|---|---|
| Data unit | Data |
| Devices | None specific (handled by software) |

---

## 5. Layer 5: Session

### Job

**Starts, manages, and ends conversations (sessions)** between two applications. It keeps track of "who is talking to whom" so conversations do not get mixed up, and can add **checkpoints** so a long transfer can resume after interruption.

| Function | Meaning |
|----------|---------|
| **Establish** | Open a session |
| **Maintain** | Keep it alive, keep dialogues in order |
| **Terminate** | Close it cleanly |
| **Recover** | Resume from a checkpoint if the connection drops |

Examples: remote procedure calls (RPC), NetBIOS, session setup in video calls, login sessions.

### What happens here in our example

The browser and server start a session: a TLS session is negotiated, and a login session or cookie may track you across several requests.

### Honest note

Like layer 6, Layer 5 is mostly **built into applications** in practice. The TCP/IP model merges layers 5, 6, and 7 into a single "Application" layer.

**Analogy:** a phone call: dialing, talking, hanging up.

| | |
|---|---|
| Data unit | Data |
| Devices | None specific |

---

## 6. Layer 4: Transport

### Job

Delivers data **from a program on one device to a program on another device**, **end to end**. It does this with **port numbers**, and decides how **reliable** the delivery should be.

### Ports

An IP address finds the **device**; a **port** finds the **program** (service) on that device.

```
203.0.113.50 : 443      -> the HTTPS service on that server
192.168.1.10 : 51514    -> the browser tab on your PC (a random "ephemeral" port)
```

| Port range | Name | Used for |
|------------|------|----------|
| 0 - 1023 | Well-known | Standard services (80, 443, 22, 53...) |
| 1024 - 49151 | Registered | Specific applications |
| 49152 - 65535 | Dynamic / ephemeral | Temporary ports chosen by clients |

**IP + port = socket.** A connection is identified by source IP, source port, destination IP, destination port.

### TCP vs UDP

| | **TCP** | **UDP** |
|---|---------|---------|
| Full name | Transmission Control Protocol | User Datagram Protocol |
| Connection | **Connection-oriented** (handshake first) | **Connectionless** (just send) |
| Reliability | **Reliable**: acknowledgements, retransmission, ordering | **Best effort**: no guarantees |
| Speed / overhead | Slower, more overhead | Faster, lighter |
| Data unit | **Segment** | **Datagram** |
| Used for | Web, email, file transfer, SSH | DNS lookups, video calls, streaming, gaming, DHCP |

### How TCP works (the essentials)

**1. Three-way handshake** (opens the connection):

```
Client                          Server
  |------- SYN ----------------->|     "I want to talk."
  |<------ SYN + ACK ------------|     "OK, I'm ready too."
  |------- ACK ----------------->|     "Great, let's go."
```

**2. Sequence numbers + acknowledgements:** every segment is numbered. The receiver confirms what arrived; anything missing is **retransmitted**, and out-of-order segments are put back in order.

**3. Flow control:** the receiver tells the sender how much data it can handle (the "window"), so a fast sender does not overwhelm a slow receiver.

**4. Segmentation:** big data is cut into pieces that fit the network, then reassembled at the other end.

**5. Closing:** the connection is ended with `FIN` / `ACK` messages.

### Note about HTTP/3

Most web traffic is HTTP over TCP. **HTTP/3** instead uses **QUIC**, which runs over **UDP** (port 443) and builds its own reliability and encryption. So not all web traffic uses TCP.

### What happens here in our example

The Layer 4 software on your PC opens a TCP connection to `203.0.113.50:443`, then cuts the (encrypted) request into segments and adds a **TCP header** with:

- source port `51514`
- destination port `443`
- sequence number, flags, window size...

**Analogy:** the delivery service that numbers the parcels, tracks them, and asks for a signature on arrival.

| | |
|---|---|
| Data unit | Segment (TCP) / Datagram (UDP) |
| Devices | Stateful firewalls, load balancers (L4) |

---

## 7. Layer 3: Network

### Job

**Logical addressing and routing**: moving packets **across different networks** from the source device to the destination device. This is the layer where the **IP address** lives (see [IPv4](ipv4-basics.md) and [IPv6](ipv6-basics.md)).

| Function | Meaning |
|----------|---------|
| **Logical addressing** | Source and destination IP addresses |
| **Routing** | Choosing the best path through many networks (routing tables) |
| **Forwarding** | Passing the packet to the next hop |
| **Fragmentation** | Splitting a packet if it is too big for a link (IPv4) |

### Key protocols

| Protocol | Purpose |
|----------|---------|
| **IPv4 / IPv6** | Addressing and delivery |
| **ICMP** | Error messages and diagnostics. **`ping` and `traceroute` use ICMP**, which has no ports. |
| **IPsec** | Secure IP communication |
| Routing protocols (OSPF, BGP, RIP) | Routers learn paths from each other |

### Key ideas

- The IP header holds the **source IP** and **destination IP**, and a **TTL (Time To Live)**. Each router subtracts 1 from TTL, and when it hits 0 the packet is dropped. This stops packets from looping forever.
- The device decides **local or remote** using the subnet mask (see [subnetting](subnetting-basics.md)). If remote, it sends to the **default gateway**.
- The **IP addresses stay the same** end to end (except where NAT rewrites them); the MAC addresses change at each hop.

### What happens here in our example

The segment becomes the payload of an IP **packet**:

```
Source IP:      192.168.1.10
Destination IP: 203.0.113.50
TTL:            64
Protocol:       TCP
```

PC A sees `203.0.113.50` is not in `192.168.1.0/24`, so it will send to the gateway `192.168.1.1`.

**Analogy:** the postal system's address and route planning: which cities the letter passes through.

| | |
|---|---|
| Data unit | Packet |
| Devices | **Routers**, Layer 3 switches |

---

## 8. Layer 2: Data Link

### Job

Delivers data **between two devices on the same local link** (the same LAN or Wi-Fi). It uses **MAC addresses**, wraps the packet in a **frame**, and checks it for errors. (This is where [ARP](arp-and-mac-basics.md) helps out.)

| Function | Meaning |
|----------|---------|
| **Physical addressing** | Source and destination **MAC** addresses |
| **Framing** | Wraps the packet with a header and trailer |
| **Error detection** | A checksum (the FCS, a CRC) in the trailer catches corrupted frames; bad frames are discarded |
| **Access control** | Decides who may use the shared medium and when (important for Wi-Fi) |

### Ethernet frame

```
+----------+----------+-----------+------------------------+-------+
| Dest MAC | Src MAC  | EtherType |   Payload (IP packet)  |  FCS  |
|  6 bytes | 6 bytes  |  2 bytes  |   up to 1500 bytes     | 4 B   |
+----------+----------+-----------+------------------------+-------+
 \_______________ header _________/ \____ data ____________/ trailer
```

The common maximum payload is **1500 bytes** (the **MTU**, Maximum Transmission Unit).

### Key technologies

| Technology | Notes |
|------------|-------|
| **Ethernet (IEEE 802.3)** | Wired LANs |
| **Wi-Fi (IEEE 802.11)** | Wireless LANs |
| **VLANs (802.1Q)** | Split one physical switch into several logical LANs |
| **ARP** | Finds the MAC for an IP (sits between L2 and L3) |
| **Switching** | Switch learns which MAC is on which port |

### Two sub-layers (just so you recognize the terms)

- **LLC** (Logical Link Control): talks to the layer above.
- **MAC** (Media Access Control): addressing and access to the medium.

### What happens here in our example

PC A needs the gateway's MAC (it gets it using ARP) and builds the frame:

```
Dest MAC:  CC:CC:CC:CC:CC:03   (the router, the next hop, NOT the far server)
Src MAC:   AA:AA:AA:AA:AA:01
Payload:   the IP packet from Layer 3
FCS:       error-check value
```

The **switch** reads the destination MAC and forwards the frame only out of the right port. At the router, this frame is **stripped off and rebuilt** with new MACs for the next link, while the IP packet inside stays the same.

**Analogy:** the local delivery van: "from this depot to the next depot".

| | |
|---|---|
| Data unit | **Frame** |
| Devices | **Switches**, bridges, wireless access points, NICs |

---

## 9. Layer 1: Physical

### Job

Moves **raw bits (0s and 1s)** as physical signals across a medium. It knows nothing about addresses or data meaning, only voltage, light, or radio waves.

| Medium | How bits are carried |
|--------|----------------------|
| **Copper cable** (twisted pair: Cat5e, Cat6) | Electrical voltage changes |
| **Fiber optic** | Pulses of light |
| **Wireless** (Wi-Fi, Bluetooth, cellular) | Radio waves |

### What it defines

| Aspect | Examples |
|--------|----------|
| **Cables and connectors** | RJ45, fiber connectors, USB |
| **Signal encoding** | How a 1 and a 0 look as voltage, light, or radio |
| **Bit rate** | 100 Mbps, 1 Gbps, 10 Gbps |
| **Topology** | Star, bus, ring (how devices are physically connected) |
| **Transmission mode** | Simplex (one way), half duplex (taking turns), full duplex (both at once) |
| **Pin layout, voltages, frequencies, max cable length** | Physical specs |

### Devices

| Device | Notes |
|--------|-------|
| **Hub** | Repeats incoming signals to **all** ports (dumb, outdated; it has no idea about MACs) |
| **Repeater** | Boosts a signal to extend distance |
| **Cables, connectors, patch panels** | The medium itself |
| **NIC** (the electrical/radio part) | Sends and receives the signals |

### What happens here in our example

The frame is converted to bits, for example `10101010...`, and the network card sends them as electrical pulses down the Ethernet cable (or as radio waves over Wi-Fi) to the switch.

**Analogy:** the road and the truck: the actual physical movement.

| | |
|---|---|
| Data unit | **Bits** |
| Devices | Cables, hubs, repeaters, NIC (physical part) |

---

## 10. Encapsulation: One Web Request Through All Layers

**Encapsulation** = each layer, going down, **wraps** the data from the layer above with its own **header** (and sometimes a trailer). On the receiving side, the reverse happens: **decapsulation**, each layer strips its own header and passes the rest up.

### Going down (sender: PC A)

```
L7  Application    [ HTTP request: GET /index.html ]
                              |
L6  Presentation   [ encrypted / encoded (TLS) ]
                              |
L5  Session        [ session established ]
                              |   (all three = "Data")
                              v
L4  Transport      [ TCP hdr | Data ]                       = Segment
                     src port 51514, dst port 443
                              |
                              v
L3  Network        [ IP hdr | TCP hdr | Data ]              = Packet
                     src 192.168.1.10, dst 203.0.113.50
                              |
                              v
L2  Data Link      [ Eth hdr | IP hdr | TCP hdr | Data | FCS ] = Frame
                     dst MAC CC:..:03, src MAC AA:..:01
                              |
                              v
L1  Physical       1 0 1 1 0 0 1 0 1 ...                    = Bits on the wire
```

### Along the way

```
 PC A --> Switch --> Router --> ... routers ... --> Server

 Switch:   reads L2 (MAC). Forwards the frame.
 Router:   removes the L2 frame, reads L3 (IP), picks next hop,
           builds a NEW L2 frame, sends on.
 Each hop: the MACs change, the IPs stay the same.
```

### Coming up (receiver: the server)

The server's network card receives the bits (L1), checks the frame (L2: is the destination MAC mine? is the FCS good?), removes the frame header, checks the IP (L3: is this IP mine?), passes the segment up, TCP (L4) checks the port `443` and reassembles the data in order, the session (L5) and decryption (L6) are handled, and finally the web server program (L7) receives `GET /index.html` and sends back `200 OK` the same way in reverse.

### Data unit names (PDUs)

| Layer | PDU name |
|-------|----------|
| 7, 6, 5 | **Data** |
| 4 | **Segment** (TCP) / **Datagram** (UDP) |
| 3 | **Packet** |
| 2 | **Frame** |
| 1 | **Bits** |

### What each network device "sees"

| Device | Looks at | Decides based on |
|--------|----------|------------------|
| Hub | Layer 1 | Nothing, copies the signal to all ports |
| **Switch** | Layer 2 | **MAC address** |
| **Router** | Layer 3 | **IP address** |
| Firewall | Layers 3 to 7 (depending on type) | IPs, ports, applications |
| Load balancer | Layer 4 or 7 | Ports, or HTTP content |

---

## 11. OSI vs TCP/IP

The Internet actually runs on the **TCP/IP model**, which has fewer layers.

| OSI (7 layers) | TCP/IP (4 layers) | Examples |
|----------------|-------------------|----------|
| 7 Application | **Application** | HTTP, DNS, SMTP, SSH |
| 6 Presentation | | TLS, encoding |
| 5 Session | | |
| 4 Transport | **Transport** | TCP, UDP |
| 3 Network | **Internet** | IP, ICMP |
| 2 Data Link | **Network Access** (Link) | Ethernet, Wi-Fi |
| 1 Physical | | Cables, radio |

(Some textbooks use a 5-layer version, with the Link and Physical layers shown separately.)

**Which one do you use?** TCP/IP describes how the real Internet is built. OSI is used to **explain, discuss, and troubleshoot**: "that's a Layer 2 issue".

---

## 12. Troubleshooting by Layer

A useful approach is to work **bottom up**, because a problem low down breaks everything above it.

| Layer | Typical problems | Things to check / tools |
|-------|------------------|-------------------------|
| **1 Physical** | Unplugged or damaged cable, dead port, Wi-Fi off, weak signal | Link lights, swap the cable, check Wi-Fi |
| **2 Data Link** | Wrong VLAN, bad switch port, duplicate MACs, ARP problems | `arp -a`, switch MAC table |
| **3 Network** | Wrong IP / mask / gateway, routing problem | `ipconfig` / `ip addr`, `ping`, `traceroute` |
| **4 Transport** | Port blocked by a firewall, service not listening | `netstat` / `ss`, `nc` or `telnet host port` |
| **5-7 Upper layers** | DNS failure, wrong URL, certificate error, app bug (404, 500) | `nslookup` / `dig`, `curl -v`, browser developer tools, Wireshark |

### A typical thought process for "I can't open a website"

1. **Is the cable or Wi-Fi connected?** (L1)
2. **Do I have a valid IP?** If it starts with `169.254` something is wrong with DHCP. (L2/L3)
3. **Can I ping my gateway?** (L3)
4. **Can I ping an outside IP like `8.8.8.8`?** Yes means routing works. (L3)
5. **Can I ping a name like `example.com`?** If not, it is a **DNS** problem. (L7)
6. **Does the site open on port 443?** If not, a firewall or server problem. (L4)
7. **Do I see an error page (404, 500)?** Then the server and application are reached, and the problem is at the application level. (L7)

---

## 13. Practice Questions

1. Name the 7 layers from top to bottom.
2. At which layer does HTTP work? Is Chrome a Layer 7 protocol?
3. What are the data units at layers 4, 3, 2 and 1?
4. At which layer do these live: a MAC address, an IP address, a port number?
5. Which layer does a **switch** work at? A **router**?
6. TCP or UDP: (a) a DNS lookup, (b) downloading a file, (c) a video call.
7. What is the TCP three-way handshake?
8. What does the TTL in an IP packet protect against?
9. When a frame crosses a router, what changes: the MAC addresses, the IP addresses, or both?
10. A user says "I have a link light on my network card but no internet; `ping 8.8.8.8` works but `ping example.com` fails." Which layer is the likely problem?
11. Which layer does encryption (TLS) belong to in the OSI teaching model? Where does it sit in practice?
12. Which OSI layers does the TCP/IP "Application" layer cover?

<details>
<summary>Show answers</summary>

1. Application, Presentation, Session, Transport, Network, Data Link, Physical (**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing).
2. HTTP is **Layer 7**. Chrome is a program, not a protocol; it *uses* HTTP.
3. L4 **segment** (TCP) / datagram (UDP); L3 **packet**; L2 **frame**; L1 **bits**.
4. MAC: **Layer 2**. IP: **Layer 3**. Port: **Layer 4**.
5. Switch: **Layer 2** (MAC). Router: **Layer 3** (IP).
6. (a) **UDP** (small, fast), (b) **TCP** (must be reliable), (c) **UDP** (speed matters more than perfect delivery).
7. **SYN**, **SYN-ACK**, **ACK**: the client and server agree to open a connection.
8. It prevents packets from looping forever: each router subtracts 1, and at 0 the packet is dropped.
9. **The MAC addresses change** (a new frame is built for the next link); the **IP addresses stay the same** (unless NAT rewrites them).
10. **Layer 7 (DNS)**. The network works (ping to an IP succeeds) but name resolution fails.
11. Teaching model: **Layer 6 (Presentation)**. In practice TLS sits between the Transport and Application layers.
12. **Layers 5, 6, and 7** (Session, Presentation, Application).

</details>

---

## 14. Quick Cheat Sheet

| # | Layer | One-line job | PDU | Identifier | Examples | Devices |
|---|-------|--------------|-----|------------|----------|---------|
| 7 | **Application** | Network services for apps | Data | URL / name | HTTP, DNS, SMTP, SSH | L7 firewall, proxy |
| 6 | **Presentation** | Format, encrypt, compress | Data | | TLS, JPEG, UTF-8 | |
| 5 | **Session** | Open, manage, close dialogues | Data | | RPC, session setup | |
| 4 | **Transport** | End-to-end delivery between programs | Segment / Datagram | **Port** | TCP, UDP | Stateful firewall |
| 3 | **Network** | Routing between networks | Packet | **IP address** | IP, ICMP | **Router** |
| 2 | **Data Link** | Delivery on the local link | Frame | **MAC address** | Ethernet, Wi-Fi, ARP | **Switch**, AP |
| 1 | **Physical** | Bits as signals | Bits | | Cables, radio, light | Hub, repeater, cable |

### Remember in one sentence

> **Going down the stack, each layer wraps your data with the information it needs (HTTP request, then ports, then IP addresses, then MAC addresses, then raw bits); going up at the receiver, each layer strips its wrapping off again.**

---

*Notes by [your name]. Feel free to open an issue or PR if you spot a mistake.*
