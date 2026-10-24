# ARP: How Devices Resolve IP to MAC (and Why MAC Never Changes)

> Beginner-friendly notes. This builds on [Networking Basics](networking-basics.md), [IPv4 Basics](ipv4-basics.md) and [Subnetting Basics](subnetting-basics.md).

## Table of Contents

1. [The Problem ARP Solves](#1-the-problem-arp-solves)
2. [What Is ARP?](#2-what-is-arp)
3. [How ARP Works, Step by Step](#3-how-arp-works-step-by-step)
4. [Talking to a Device on Another Network](#4-talking-to-a-device-on-another-network)
5. [The ARP Table (ARP Cache)](#5-the-arp-table-arp-cache)
6. [Why a MAC Address Never Changes](#6-why-a-mac-address-never-changes)
7. [ARP Security Risks](#7-arp-security-risks)
8. [Hands-On Lab](#8-hands-on-lab)
9. [Practice Questions](#9-practice-questions)
10. [Quick Cheat Sheet](#10-quick-cheat-sheet)

---

## 1. The Problem ARP Solves

Quick recap of the two addresses:

| | IP address | MAC address |
|---|-----------|-------------|
| Layer | 3 (Network) | 2 (Data Link) |
| Example | `192.168.1.20` | `BB:BB:BB:BB:BB:02` |
| Used for | Finding the right **network** and final destination | Delivering a frame to the right device **on the local wire/Wi-Fi** |

Applications only know the **IP address** (you type `192.168.1.20` or a name that resolves to an IP).

But the network card can only put a frame on the wire if the frame has a **destination MAC address**. A switch does not understand IPs at all; it only reads MACs.

So before sending anything on a LAN, a device must answer:

> "I know the IP address. What is the **MAC address** that goes with it?"

That translation job is done by **ARP**.

---

## 2. What Is ARP?

**ARP = Address Resolution Protocol.** It maps a known **IPv4 address** to its **MAC address** on the **local network**.

Key points:

- It works **only inside one LAN** (one broadcast domain). ARP messages do not cross routers.
- It sits between layer 2 and layer 3, so it is sometimes called a "layer 2.5" protocol.
- The **request** is a **broadcast** (sent to everyone on the LAN).
- The **reply** is a **unicast** (sent only back to the asker).
- ARP frames use EtherType `0x0806` (just so you recognize it in Wireshark).

**Analogy:** you are in a big hall and you know a person's seat number (IP) but need their name (MAC). You shout: *"Who is in seat 20? Come tell me your name!"* Everyone hears it, but only the person in seat 20 answers.

### What an ARP message contains

| Field | Meaning |
|-------|---------|
| Opcode | `1` = request, `2` = reply |
| Sender MAC | MAC of the device sending this ARP message |
| Sender IP | IP of the device sending this ARP message |
| Target MAC | In a request: unknown, filled with `00:00:00:00:00:00`. In a reply: the answer. |
| Target IP | The IP we want to resolve |

---

## 3. How ARP Works, Step by Step

Setup used in all examples:

| Device | IP | MAC |
|--------|----|-----|
| PC A | 192.168.1.10 | `AA:AA:AA:AA:AA:01` |
| PC B | 192.168.1.20 | `BB:BB:BB:BB:BB:02` |
| Router (LAN side) | 192.168.1.1 | `CC:CC:CC:CC:CC:03` |

All have mask `255.255.255.0` and are connected through one switch.

**Goal:** PC A wants to ping PC B (`192.168.1.20`).

### Step 1: Same network?

PC A uses its own IP and mask to check: is `192.168.1.20` in my network `192.168.1.0/24`? **Yes** -> deliver directly, no router needed.

### Step 2: Check the ARP cache

PC A looks in its **ARP table** for `192.168.1.20`. If an entry exists, skip to Step 6. Suppose it is **empty**.

### Step 3: Send an ARP request (broadcast)

PC A broadcasts to the whole LAN:

```
Ethernet header:  src MAC = AA:AA:AA:AA:AA:01
                  dst MAC = FF:FF:FF:FF:FF:FF   <- broadcast, "everyone"
ARP message:      "Who has 192.168.1.20? Tell 192.168.1.10"
```

The switch **floods** this frame out of every port (it is a broadcast).

### Step 4: Everyone checks, only one answers

- PC B sees its own IP and responds.
- The router and everyone else see it is not for them and ignore it.

### Step 5: ARP reply (unicast)

```
Ethernet header:  src MAC = BB:BB:BB:BB:BB:02
                  dst MAC = AA:AA:AA:AA:AA:01   <- straight back to PC A
ARP message:      "192.168.1.20 is at BB:BB:BB:BB:BB:02"
```

### Step 6: Cache it and send the real data

PC A saves `192.168.1.20 -> BB:BB:BB:BB:BB:02` in its ARP table, then sends the actual ICMP ping in a frame with destination MAC `BB:BB:BB:BB:BB:02`.

The switch now forwards that frame **only** to PC B's port (it learned MAC-to-port from the ARP reply's source MAC).

### The whole conversation as a picture

```
 PC A                      Switch                      PC B
  |  ARP request (broadcast) |                           |
  |------------------------->|-------------------------->|
  |                          |   (flooded to all ports)  |
  |                          |                           |
  |               ARP reply (unicast)                    |
  |<-------------------------|<--------------------------|
  |                          |                           |
  |  [A saves B's MAC in its ARP table]                  |
  |                          |                           |
  |  Real data to MAC BB:...:02                          |
  |------------------------->|-------------------------->|
```

PC B also usually learns PC A's MAC from the request, so it does not need to ARP back later.

---

## 4. Talking to a Device on Another Network

**Important rule: you only ARP for a device on your own LAN.** You can never ARP for a device on the other side of a router.

Goal: PC A (`192.168.1.10/24`) wants to reach `8.8.8.8`.

1. PC A checks: is `8.8.8.8` in `192.168.1.0/24`? **No.**
2. So it sends the packet to its **default gateway** (`192.168.1.1`, the router).
3. PC A **ARPs for the gateway's IP**, not for `8.8.8.8`, and gets `CC:CC:CC:CC:CC:03`.
4. The frame leaves with **destination MAC = router**, but **destination IP = 8.8.8.8**.
5. The router removes the old frame, looks at the destination **IP**, decides the next hop, and builds a **new frame** with new MAC addresses for the next link (doing its own ARP on that side if needed).

### What changes and what stays the same

| Hop | Src MAC | Dst MAC | Src IP | Dst IP |
|-----|---------|---------|--------|--------|
| PC A to Router | `AA:AA:AA:AA:AA:01` | `CC:CC:CC:CC:CC:03` | 192.168.1.10 | 8.8.8.8 |
| Router to ISP router | Router's WAN MAC | ISP router's MAC | Router's public IP (*) | 8.8.8.8 |
| ISP router to next router | ISP router's MAC | Next router's MAC | same as previous | 8.8.8.8 |

(*) Your home router's **NAT** replaces your private source IP with its public one. Aside from NAT, **the IP addresses stay the same end to end while the MAC addresses change at every hop.**

> **Memory trick:** MAC = "who is the **next stop**?" IP = "who is the **final destination**?"

---

## 5. The ARP Table (ARP Cache)

Every device keeps a small table of recently learned **IP -> MAC** pairs, so it does not have to broadcast every time.

Without it, every single packet would need an ARP broadcast first, which would be very noisy and slow.

### Viewing it

| OS | Command |
|----|---------|
| Windows, macOS, Linux (older tools) | `arp -a` |
| Linux (modern) | `ip neigh` (or `ip neighbour show`) |

Example (Windows):

```
Interface: 192.168.1.10 --- 0x5
  Internet Address      Physical Address      Type
  192.168.1.1           cc-cc-cc-cc-cc-03     dynamic
  192.168.1.20          bb-bb-bb-bb-bb-02     dynamic
  192.168.1.255         ff-ff-ff-ff-ff-ff     static
```

Example (Linux):

```
192.168.1.1  dev wlan0 lladdr cc:cc:cc:cc:cc:03 REACHABLE
192.168.1.20 dev wlan0 lladdr bb:bb:bb:bb:bb:02 STALE
```

### Dynamic vs static entries

| Type | Meaning |
|------|---------|
| **Dynamic** | Learned automatically through ARP. Expires after a while if unused. |
| **Static** | Added manually or built in (like the broadcast entry). Does not expire. |

### Why entries expire

Devices can change: a laptop can leave, an IP can be given to a different device, a network card can be replaced. Old entries are removed after a timeout so the table does not hold wrong information. The timeout depends on the system: typically **seconds to a few minutes on PCs** and **4 hours by default on Cisco routers**.

### Related table: the switch's MAC address table

Do not confuse the two tables:

| | ARP table | MAC address table (CAM table) |
|---|-----------|-------------------------------|
| Lives on | Hosts and routers | Switches |
| Maps | **IP -> MAC** | **MAC -> switch port** |
| Used to | Know whom to address a frame to | Know which port to send a frame out of |

---

## 6. Why a MAC Address Never Changes

### The short answer

A MAC address belongs to the **network card (interface)**, not to the **network** you are connected to. It is assigned at the factory, so moving to a different Wi-Fi does not change it.

### Structure of a MAC address

48 bits, written as 12 hex digits:

```
  A4:5E:60 : C1:2B:9F
  \______/   \______/
    OUI      Device-specific
 (vendor)    (serial-like part)
```

- **First 24 bits (OUI):** identifies the manufacturer (Apple, Intel, Cisco...).
- **Last 24 bits:** assigned by that manufacturer, so the pair should be globally unique.
- Also called the **burned-in address (BIA)** or **physical address**.

### Why it is designed this way

| MAC address | IP address |
|-------------|------------|
| Identifies **the hardware** | Identifies **where the device is connected** |
| Fixed identity ("who you are") | Changes when you change network ("where you are") |
| Only matters on the local link | Has to work across the whole Internet |

Because MAC addresses only need to be unique and only work on the local segment, there is no reason for them to depend on location. Letting the IP carry the "where" and the MAC carry the "who" keeps things simple.

**Analogy:** your **name** is the same wherever you travel (MAC). Your **hotel room number** changes at every hotel (IP).

### "But I read that MAC changes at every hop!"

That is a different thing, and the wording confuses many beginners:

| Statement | Is it true? | Explanation |
|-----------|-------------|-------------|
| "A device's own MAC address does not change as it moves between networks" | **True** (normally) | The hardware identity stays the same |
| "The MAC addresses *in a frame's header* change at every hop" | **True** | Each link has a new sender and receiver, so each new frame uses the MACs of that link |

Think of the packet as a parcel: the **IP** is the final address on the parcel, and the **MAC addresses** are the "this truck, to the next depot" labels that get replaced at every stop. The trucks themselves (the devices) keep their identity.

### The honest fine print: "never" is not 100%

| Situation | What happens |
|-----------|--------------|
| **MAC spoofing / changing** | The OS can be told to use a different MAC. The factory-burned one still exists in hardware. |
| **Randomized MACs** | Modern phones and laptops (iOS, Android, Windows, etc.) can use a different, random MAC for each Wi-Fi network for privacy. These addresses have the "locally administered" bit set (the 2nd hex digit is `2`, `6`, `A`, or `E`). |
| **Virtual machines and containers** | Their MACs are generated by software and can differ per VM. |
| **Replacing the network card** | A new card means a new MAC. |

So a better statement is: **a physical network card's built-in MAC address does not change, and it does not depend on which network you join.**

---

## 7. ARP Security Risks

ARP was designed in a trusting era. It has **no authentication**: a device will believe any ARP reply, even one it never asked for.

### ARP spoofing (poisoning)

A malicious device on the same LAN sends fake ARP replies, claiming "the router's IP is at *my* MAC". Victims update their ARP table with the wrong MAC and start sending their traffic to the attacker. This can enable **man-in-the-middle** attacks or cause an outage.

Signs that something may be off:

- Two different IPs showing the **same MAC** in your ARP table (can also be normal, for example a router that owns several IPs).
- The gateway's MAC suddenly changes.
- Duplicate IP warnings.

### Common defenses

| Defense | Where |
|---------|-------|
| **Dynamic ARP Inspection (DAI)** | Managed switches check ARP messages against a trusted table (the DHCP snooping database) |
| **Port security** | Limit which/how many MACs are allowed on a switch port |
| **Static ARP entries** | For critical devices like the gateway (hard to manage at scale) |
| **Encrypted protocols (HTTPS, SSH, VPN)** | Even if traffic is intercepted, it is unreadable |
| **Only join networks you trust** | ARP attacks require being on the same LAN |

### Other ARP terms you may see

| Term | Meaning |
|------|---------|
| **Gratuitous ARP** | A device announces its own IP-to-MAC mapping without being asked. Used to update others' caches and to detect duplicate IPs. Common after an IP change or failover. |
| **Proxy ARP** | A router answers ARP requests on behalf of a device on another network. |
| **RARP** | Reverse ARP (MAC -> IP). Obsolete; replaced by BOOTP and DHCP. |

> **IPv6 note:** IPv6 does not use ARP. It uses **Neighbor Discovery Protocol (NDP)** for the same job.

---

## 8. Hands-On Lab

Watch ARP work on your own computer. You need two devices on the same network (or your PC and your router).

1. **Look at the current table**
   ```
   arp -a              # Windows / macOS / Linux
   ip neigh            # Linux
   ```
2. **Find your router's IP** (the default gateway)
   ```
   ipconfig            # Windows (look for "Default Gateway")
   ip route            # Linux
   ```
3. **Ping another device on the LAN** (for example your phone or the router)
   ```
   ping 192.168.1.1
   ```
4. **Look at the table again**, a new entry should be there. That is ARP at work.
5. **Clear it and watch it come back** (needs admin / root)
   ```
   arp -d *                    # Windows (run as Administrator)
   sudo ip neigh flush all     # Linux
   ```
   Then ping again and re-check.
6. **Optional: capture it in Wireshark.** Use the display filter `arp` and ping a new device. You will see the *"Who has ...? Tell ..."* request and the *"... is at ..."* reply.

Try pinging something on the Internet (like `8.8.8.8`). Check the ARP table: you will see your **gateway**, not `8.8.8.8`. This proves section 4.

---

## 9. Practice Questions

1. What does ARP map to what?
2. Is an ARP request unicast or broadcast? What about the reply?
3. What destination MAC does an ARP request use?
4. PC A (`192.168.1.10/24`) pings `10.5.5.5`. Which IP address does PC A send an ARP request for?
5. Why is the second ping to the same LAN host often free of ARP traffic?
6. Can ARP broadcasts cross a router?
7. What is the difference between the ARP table and the switch's MAC address table?
8. A laptop moves from home Wi-Fi to a coffee shop Wi-Fi. What stays the same and what changes: its (a) IP, (b) built-in MAC?
9. Which protocol replaces ARP in IPv6?
10. Your ARP table shows two different IPs with the same MAC. Name one harmless and one worrying explanation.

<details>
<summary>Show answers</summary>

1. A known **IPv4 address** to a **MAC address**.
2. The request is a **broadcast**; the reply is a **unicast**.
3. `FF:FF:FF:FF:FF:FF`.
4. The **default gateway's IP** (for example `192.168.1.1`). `10.5.5.5` is not on the local network, so it is never ARPed directly.
5. The result is stored in the **ARP cache**, so no new request is needed until the entry expires.
6. **No.** Routers do not forward broadcasts, so ARP stays within one LAN.
7. ARP table: **IP -> MAC**, on hosts/routers. MAC table: **MAC -> port**, on switches.
8. (a) IP **changes** (new network). (b) The built-in MAC **stays the same** (unless the device uses a randomized per-network MAC).
9. **NDP (Neighbor Discovery Protocol).**
10. Harmless: one device (such as a router or server) owns several IP addresses, or proxy ARP is in use. Worrying: ARP spoofing, where an attacker claims other IPs as its own.

</details>

---

## 10. Quick Cheat Sheet

| Concept | Remember |
|---------|----------|
| ARP | Maps IP -> MAC on the **local network only** |
| Request | **Broadcast** to `FF:FF:FF:FF:FF:FF`: "Who has this IP?" |
| Reply | **Unicast**: "That IP is at this MAC" |
| Remote destination | ARP for the **default gateway**, never for the far host |
| ARP table | Cache of recent IP -> MAC pairs (`arp -a`, `ip neigh`) |
| Switch MAC table | MAC -> port (a different thing) |
| MAC address | 48-bit hardware ID: vendor (24 bits) + device (24 bits) |
| MAC vs IP | MAC = **who** (hardware), IP = **where** (network) |
| MAC in frame headers | Changes **every hop**; the device's own MAC does not |
| Security | ARP has no authentication, so it can be spoofed |
| IPv6 | Uses NDP instead of ARP |

### Remember in one sentence

> **ARP lets a device turn an IP address into a MAC address on its own LAN; the MAC stays with the hardware, while the IP follows the network you join.**

---

*Notes by [your name]. Feel free to open an issue or PR if you spot a mistake.*
