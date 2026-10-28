# Networking Basics: LAN vs WAN, Switch vs Router, and Addresses

> Beginner-friendly notes. No prior knowledge needed.

## Table of Contents

1. [LAN vs WAN](#1-lan-vs-wan)
2. [Switch vs Router](#2-switch-vs-router)
3. [Why Devices Need Addresses](#3-why-devices-need-addresses)
4. [Putting It All Together](#4-putting-it-all-together)
5. [Quick Cheat Sheet](#5-quick-cheat-sheet)

---

## 1. LAN vs WAN

### LAN (Local Area Network)

A **LAN** is a network that covers a **small area**: a home, an office, a school lab.

- Devices in the same LAN are physically close to each other.
- You usually own and control it (your Wi-Fi at home).
- It is **fast** and **cheap** to run.

**Examples:** your home Wi-Fi, an office network, a college computer lab.

### WAN (Wide Area Network)

A **WAN** is a network that covers a **large area**: cities, countries, even the whole world.

- It connects many LANs together.
- It is usually owned by **telecom companies / ISPs** (Internet Service Providers), not by you.
- You pay to use it.

**Example:** the **Internet** is the biggest WAN in the world.

### Simple analogy

| Real life | Networking |
|-----------|------------|
| Your house | A LAN |
| The roads and highways between houses and cities | A WAN |
| Your ISP | The company that built and maintains those roads |

### Comparison

| Feature | LAN | WAN |
|---------|-----|-----|
| Area covered | Small (home, office) | Huge (cities, countries) |
| Speed | Usually very fast | Usually slower |
| Owned by | You / your organization | ISPs / telecom companies |
| Cost | Low | High |
| Example | Home Wi-Fi | The Internet |

### Picture

```
  [Home LAN]                                  [Office LAN]
  Phone  Laptop                               PC   Printer
     \    /                                     \    /
    [Router] ======= WAN (Internet / ISP) ======= [Router]
```

---

## 2. Switch vs Router

Both are boxes with cables plugged in, but they do **different jobs**.

### Switch: connects devices *inside* one network

A **switch** connects devices within the **same LAN** and lets them talk to each other.

- It keeps a table called the **MAC address table**: "Device A is on port 1, Device B is on port 2..."
- When data arrives, it looks at the **MAC address** and sends it **only to the right port**, not to everyone.
- It works at **Layer 2** (Data Link layer) of the OSI model.

**Analogy:** a switch is like the **internal mail room of a building**. It delivers letters between rooms inside the same building.

### Router: connects *different* networks

A **router** connects **different networks** together, for example your home LAN to the Internet (WAN).

- It looks at the **IP address** to decide where to send data next.
- It keeps a **routing table** that holds the best path to reach each network.
- It works at **Layer 3** (Network layer) of the OSI model.
- It often also gives out IP addresses (DHCP) and hides your private devices behind one public IP (NAT).

**Analogy:** a router is like the **post office**. It sends letters from your building to other buildings, cities, and countries.

### Comparison

| Feature | Switch | Router |
|---------|--------|--------|
| Main job | Connect devices **within** a network | Connect **different** networks |
| Uses | MAC addresses | IP addresses |
| OSI layer | Layer 2 | Layer 3 |
| Decides using | MAC address table | Routing table |
| Connects to the Internet? | No (on its own) | Yes |
| Analogy | Mail room inside a building | Post office between cities |

### Picture

```
 Phone   Laptop   PC
    \      |      /
     \     |     /
      [   Switch   ]       <- same LAN: devices talk to each other
            |
        [  Router  ]       <- gateway out of the LAN
            |
        (Internet / WAN)
```

### A note about your home "router"

The box your ISP gives you is usually **several devices in one**:

- **Router** (connects you to the internet)
- **Switch** (the LAN ports on the back)
- **Wireless access point** (Wi-Fi)
- Often a **modem** too

That is why people mix up the words. They are separate *functions* that happen to live in one box.

---

## 3. Why Devices Need Addresses

Imagine mailing a letter with no address on it. Nobody would know where to deliver it.
Networks work the same way. Every piece of data must say **who it is for** and **who it is from**.

Devices use **two kinds of addresses**.

### MAC Address: the "name" of the network card

- Example: `A4:5E:60:C1:2B:9F`
- Burned into the network card (Wi-Fi / Ethernet) by the manufacturer.
- Used for delivery **inside the same LAN**.
- Used by **switches**.
- Think of it as your **permanent name**.

### IP Address: the "location" of the device

- Example: `192.168.1.10`
- Assigned by the network (usually by the router using DHCP), so it can change.
- Used for delivery **across different networks**.
- Used by **routers**.
- Think of it as your **current home address**.

### Name vs address

| | MAC Address | IP Address |
|---|-------------|------------|
| Analogy | Your name | Your home address |
| Changes? | Normally no | Yes, depends on the network you join |
| Works in | Same LAN | Across networks and the Internet |
| Used by | Switch | Router |

### Private vs public IP addresses

- **Private IP** (works only inside your LAN): `192.168.x.x`, `10.x.x.x`, `172.16.x.x` to `172.31.x.x`
- **Public IP** (visible on the Internet): given to your router by your ISP

Many homes have the **same** private addresses (like `192.168.1.10`). That is fine, because they only matter inside each home. The router translates them to one public IP using **NAT**.

### Why not just use one type of address?

- MAC addresses are **flat**: they say *who* you are but not *where* you are. Nobody could keep a list of every MAC address on Earth.
- IP addresses are **structured**: part of the address tells which network it belongs to, so routers can forward data step by step toward the right network.

---

## 4. Putting It All Together

### Example 1: Laptop sends a file to a printer (same LAN)

1. Laptop knows the printer's IP address.
2. Laptop asks the LAN: *"Who has this IP? Tell me your MAC address."* This is called **ARP**.
3. Printer replies with its MAC address.
4. Laptop sends the data to that MAC address.
5. The **switch** reads the MAC address and sends the data only to the printer's port.

No router needed.

### Example 2: Laptop opens a website (different network)

1. The website's IP is **not** in your LAN, so the laptop sends the data to the **default gateway** (your router).
2. The **switch** delivers it to the router using its MAC address.
3. The **router** reads the destination **IP address**, checks its routing table, and forwards it out to the ISP (WAN).
4. Routers across the internet pass it along, hop by hop, until it reaches the website's server.
5. The reply comes back the same way.

```
Laptop --> Switch --> Router --> ISP --> ... --> Website server
  (MAC)     (MAC)     (IP)       (IP)    (IP)
```

**Key idea:** MAC addresses handle each **local hop**; the IP address stays the same for the whole journey and tells the data where it is ultimately going.

---

## 5. Quick Cheat Sheet

| Term | One-line meaning |
|------|------------------|
| **LAN** | Small network in one place (home, office) |
| **WAN** | Big network connecting many LANs (the Internet) |
| **Switch** | Connects devices inside a LAN using MAC addresses |
| **Router** | Connects different networks using IP addresses |
| **MAC address** | Permanent hardware ID, used inside a LAN |
| **IP address** | Logical address, used across networks |
| **Default gateway** | The router a device uses to leave its LAN |
| **ARP** | Finds the MAC address that belongs to an IP address |
| **DHCP** | Automatically gives devices IP addresses |
| **NAT** | Lets many private devices share one public IP |

### Remember in one sentence

> A **switch** moves data **inside** a network, a **router** moves data **between** networks, and **addresses** tell the data exactly where to go.

---

*Notes by Sandeep Kumar. Feel free to open an issue or PR if you spot a mistake.*
