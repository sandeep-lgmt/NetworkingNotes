# IPv6: Why It Exists, Structure, and Address Format

> Beginner-friendly notes. This builds on [IPv4 Basics](ipv4-basics.md) and [ARP and MAC Basics](arp-and-mac-basics.md). Earlier notes: [Networking Basics](networking-basics.md), [Subnetting Basics](subnetting-basics.md).

## Table of Contents

1. [Why IPv6 Exists](#1-why-ipv6-exists)
2. [Hex Basics (You Need This First)](#2-hex-basics-you-need-this-first)
3. [IPv6 Structure](#3-ipv6-structure)
4. [Shortening Rules](#4-shortening-rules)
5. [Prefix and Interface ID](#5-prefix-and-interface-id)
6. [Types of IPv6 Addresses](#6-types-of-ipv6-addresses)
7. [How Devices Get an IPv6 Address](#7-how-devices-get-an-ipv6-address)
8. [IPv4 vs IPv6 at a Glance](#8-ipv4-vs-ipv6-at-a-glance)
9. [IPv4 and IPv6 Living Together](#9-ipv4-and-ipv6-living-together)
10. [Hands-On Lab](#10-hands-on-lab)
11. [Practice Questions](#11-practice-questions)
12. [Quick Cheat Sheet](#12-quick-cheat-sheet)

---

## 1. Why IPv6 Exists

### The problem: IPv4 ran out

An IPv4 address is 32 bits, which gives about **4.3 billion** addresses (2^32). That sounded like plenty in the 1980s. Then came PCs, phones, tablets, smart TVs, cameras, cars, and watches. Today there are far more connected devices than IPv4 addresses.

- The global pool of unassigned IPv4 blocks ran out in **2011**.
- The regional registries (the organizations that hand addresses out to ISPs) have since run out or nearly run out of free blocks as well.

### The patches that kept IPv4 alive

| Workaround | What it does | Downside |
|------------|--------------|----------|
| **CIDR** | Gives out exactly-sized blocks instead of whole classes | Only slows the problem |
| **Private IPs + NAT** | Many devices share one public IP | Breaks direct "end-to-end" connections; makes hosting, peer-to-peer, and some apps harder |

These bought time, but they are workarounds, not a fix.

### The real fix: IPv6

IPv6 uses **128-bit** addresses:

```
IPv4:  2^32  = 4,294,967,296 addresses            (about 4.3 billion)
IPv6:  2^128 = 340,282,366,920,938,463,463,374,607,431,768,211,456
             = about 3.4 x 10^38 addresses
```

That is enough for every device on Earth to have its own public address, many times over.

### Other improvements

- **Simpler, fixed-size header** (40 bytes), so routers process packets faster.
- **No broadcast.** It uses multicast instead, which is less noisy.
- **Auto-configuration built in** (SLAAC): devices can get an address without a DHCP server.
- **No ARP.** It uses NDP (Neighbor Discovery Protocol) instead.
- **No NAT needed**, so devices can connect end to end again.
- **Easier subnetting**: the standard LAN size is always `/64`.
- **Designed with IPsec in mind** for secure communication.

### Fun fact: what happened to IPv5?

The version number "5" was already used for an experimental streaming protocol (ST), which never became a standard. So the successor to IPv4 jumped straight to **6**.

---

## 2. Hex Basics (You Need This First)

IPv4 uses decimal numbers in each octet. IPv6 is written in **hexadecimal (base 16)** because 128 bits would be unreadable in decimal or binary.

Hex has **16 digits**: `0-9` and `a-f`.

| Hex | Decimal | Binary | | Hex | Decimal | Binary |
|-----|---------|--------|---|-----|---------|--------|
| 0 | 0 | 0000 | | 8 | 8 | 1000 |
| 1 | 1 | 0001 | | 9 | 9 | 1001 |
| 2 | 2 | 0010 | | a | 10 | 1010 |
| 3 | 3 | 0011 | | b | 11 | 1011 |
| 4 | 4 | 0100 | | c | 12 | 1100 |
| 5 | 5 | 0101 | | d | 13 | 1101 |
| 6 | 6 | 0110 | | e | 14 | 1110 |
| 7 | 7 | 0111 | | f | 15 | 1111 |

**Key idea:** **one hex digit = exactly 4 bits** (a "nibble"). So:

- 4 hex digits = 16 bits
- 32 hex digits = 128 bits (a full IPv6 address)

Example: `ff` = `1111 1111` = 255, `10` hex = `0001 0000` = 16 decimal.

Hex is not case sensitive (`A` = `a`), but **lowercase is the recommended style** for IPv6.

---

## 3. IPv6 Structure

An IPv6 address is:

- **128 bits** long
- Split into **8 groups** of **16 bits** each
- Each group is written as **4 hex digits**
- Groups are separated by **colons** (`:`)

Each group is called a **hextet** (or sometimes a "block" or "field"; "hextet" is not an official term, but it is widely used).

```
 2001 : 0db8 : 85a3 : 0000 : 0000 : 8a2e : 0370 : 7334
  |      |      |      |      |      |      |      |
 16 bits each  x  8 groups  =  128 bits
```

The full, unshortened form (all 32 hex digits) is called the **expanded** or **long form**.

### IPv4 vs IPv6 notation

```
IPv4:   192.168.1.10                                      4 numbers, dots, decimal
IPv6:   2001:0db8:85a3:0000:0000:8a2e:0370:7334           8 groups, colons, hex
```

---

## 4. Shortening Rules

Writing 32 hex digits is painful, so there are **two shortening rules**.

### Rule 1: Drop leading zeros in each group

Zeros at the **start** of a group can be removed. At least one digit must remain in each group.

```
0db8  ->  db8
0370  ->  370
0000  ->  0
```

Only **leading** zeros. You can never drop zeros at the end: `0370` becomes `370`, **not** `37`.

### Rule 2: Replace one run of all-zero groups with `::`

One run of consecutive groups that are all zero can be replaced by a double colon `::`.

**The `::` can only be used once in an address**, otherwise nobody could tell how many zero groups each one stands for.

### Putting it together

```
Full:        2001:0db8:85a3:0000:0000:8a2e:0370:7334
Rule 1:      2001:db8:85a3:0:0:8a2e:370:7334
Rule 2:      2001:db8:85a3::8a2e:370:7334
```

```
Full:        2001:0db8:0000:0000:0000:0000:0000:0001
Rule 1:      2001:db8:0:0:0:0:0:1
Rule 2:      2001:db8::1
```

### Which zeros get the `::`?

If there are **several** zero runs, shorten the **longest** one. If they are the same length, shorten the **first** one. Do not use `::` for a single zero group (write `0` instead).

```
Full:     2001:0db8:0000:0001:0000:0000:0000:0001
Short:    2001:db8:0:1::1
          (the run of three zero groups gets the ::; the single zero stays as 0)
```

### How to expand a short address back to full

1. Count the groups you can see.
2. `8 - (groups you see)` = how many all-zero groups `::` stands for.
3. Insert them where `::` is, then pad every group to 4 digits.

```
2001:db8::1
Visible groups: 2001, db8, 1   -> 3 groups
Missing: 8 - 3 = 5 zero groups

2001:0db8:0000:0000:0000:0000:0000:0001
```

### Invalid addresses (common mistakes)

| Address | Why it is invalid |
|---------|-------------------|
| `2001::db8::1` | Two `::`, so it is ambiguous |
| `2001:db8:g::1` | `g` is not a hex digit |
| `2001:db8:12345::1` | A group has 5 digits (max 4) |
| `2001:db8:1:2:3:4:5:6:7` | 9 groups (must be 8) |

### Two special short addresses

| Address | Meaning |
|---------|---------|
| `::1` | **Loopback**, this device itself (like IPv4 `127.0.0.1`) |
| `::` | **Unspecified**, "no address yet" (like IPv4 `0.0.0.0`) |

---

## 5. Prefix and Interface ID

Just like IPv4, an IPv6 address has a **network part** and a **host part**.

```
|<------------- 64 bits ------------->|<------------- 64 bits ------------->|
|            Network prefix           |          Interface ID                |
|  (which network / subnet)           |  (which device on that network)      |
```

- **Prefix** (first part): identifies the network. Handed down from your ISP.
- **Interface ID** (last 64 bits): identifies the device on that network.

### Prefix length notation

IPv6 uses the **same slash notation as CIDR**:

```
2001:db8:abcd:12::/64
```

`/64` means "the first 64 bits are the network prefix". There are no dotted subnet masks in IPv6; only the slash.

### How an ISP's allocation is divided

```
|   48 bits: given by ISP   | 16 bits: subnet ID |      64 bits: interface ID     |
|  2001:0db8:abcd           |       0012         |  0000:0000:0000:0005           |
```

So if you receive a `/48`:

- You have **16 bits** to make subnets: 2^16 = **65,536 subnets**, each of which is a `/64`.
- A home user is often given a `/56` (2^8 = **256** subnets) or `/64`.

### Subnetting in IPv6 is much easier

- Every LAN is a **/64**. Always. (This is required for features like SLAAC to work.)
- You do not count hosts. A `/64` has 2^64 (about 18 quintillion) addresses.
- **No network address and no broadcast address to subtract.** There is no "minus 2" rule here.
- Subnetting means incrementing the **subnet ID** group:

```
2001:db8:abcd:0001::/64   <- subnet 1
2001:db8:abcd:0002::/64   <- subnet 2
2001:db8:abcd:0003::/64   <- subnet 3
```

Compare this with the block-size math in the [subnetting notes](subnetting-basics.md).

### Finding the prefix of an address

For `2001:db8:abcd:12::5/64`, keep the first 64 bits (the first 4 groups) and zero the rest:

```
Prefix:  2001:db8:abcd:12::/64
```

---

## 6. Types of IPv6 Addresses

IPv6 has three types: **unicast**, **multicast**, and **anycast**. **There is no broadcast.**

| Type | Meaning |
|------|---------|
| **Unicast** | One sender to **one** specific interface |
| **Multicast** | One sender to **a group** of interfaces that joined it (replaces broadcast) |
| **Anycast** | One sender to the **nearest** of several devices sharing the same address (used for things like DNS services) |

### Common address ranges

| Prefix | Name | Looks like | IPv4 equivalent / meaning |
|--------|------|-----------|---------------------------|
| `2000::/3` | **Global unicast** | starts with `2` or `3` | **Public IP**, routable on the Internet |
| `fe80::/10` | **Link-local** | `fe80::...` | Valid on **one link only**, never routed. Every IPv6 interface has one automatically. Used for NDP and talking to the router. (Similar in spirit to IPv4 `169.254.x.x`, but always present.) |
| `fc00::/7` (in practice `fd00::/8`) | **Unique local** | `fd..` | **Private IP**, like `192.168.x.x` |
| `ff00::/8` | **Multicast** | `ff..` | Replaces broadcast |
| `::1/128` | Loopback | `::1` | Like `127.0.0.1` |
| `::/128` | Unspecified | `::` | Like `0.0.0.0` |
| `2001:db8::/32` | **Documentation** | `2001:db8:...` | Reserved for examples and tutorials, like this one. Never used on the real Internet. |

### Useful multicast addresses

| Address | Meaning |
|---------|---------|
| `ff02::1` | All nodes on the local link (what broadcast used to do) |
| `ff02::2` | All routers on the local link |

### Quick identification trick

Look at the **first few characters**:

```
2xxx: or 3xxx:   -> global (public)
fe80:            -> link-local
fdxx:            -> unique local (private)
ff..:            -> multicast
::1              -> loopback
```

### A device usually has several IPv6 addresses

At the same time, one network interface typically has:

- a **link-local** address (`fe80::...`)
- one or more **global** addresses (`2001:...`)
- possibly temporary privacy addresses

This is normal and not a problem.

---

## 7. How Devices Get an IPv6 Address

| Method | How it works |
|--------|--------------|
| **SLAAC** (Stateless Address Autoconfiguration) | The router announces the network prefix in a **Router Advertisement**. The device adds its own 64-bit interface ID to build its address. No server needed. |
| **DHCPv6** | A server hands out addresses, like DHCP in IPv4 (and can provide extra settings such as DNS) |
| **Static** | Set manually, usually for servers and routers |

### Where does the interface ID come from?

Two common ways:

**1. EUI-64 (derived from the MAC address)**

Take the 48-bit MAC, insert `ff:fe` in the middle, then flip the 7th bit of the first byte.

```
MAC:               00:1a:2b:3c:4d:5e
Insert ff:fe:      00:1a:2b:ff:fe:3c:4d:5e
Flip 7th bit
(00 -> 02):        02:1a:2b:ff:fe:3c:4d:5e
Interface ID:      021a:2bff:fe3c:4d5e   ->  21a:2bff:fe3c:4d5e (shortened)

Full address with prefix 2001:db8:1:1::/64:
2001:db8:1:1:21a:2bff:fe3c:4d5e
```

This links back to the [MAC notes](arp-and-mac-basics.md): the MAC identifies the hardware, so an address built from it never changes, which **lets you be tracked across networks**.

**2. Random / temporary (privacy) addresses**

Modern operating systems usually generate a **random** interface ID (and rotate temporary ones) so you cannot be tracked by a fixed value. This is the IPv6 counterpart of the randomized MAC addresses on phones.

### "No NAT" does not mean "no firewall"

With IPv6 every device can have a public address, but that does **not** mean every device is open to the world. Home routers include a **firewall** that blocks unsolicited incoming connections by default. NAT was never a real security feature; the firewall is.

---

## 8. IPv4 vs IPv6 at a Glance

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address size | 32 bits | 128 bits |
| Notation | Dotted decimal (`192.168.1.10`) | Colon-separated hex (`2001:db8::1`) |
| Address count | ~4.3 billion | ~3.4 x 10^38 |
| Groups | 4 octets | 8 hextets |
| Header size | 20 bytes minimum, variable | 40 bytes, fixed |
| Subnet notation | Mask or `/n` | `/n` only |
| Typical LAN size | Any (`/24`, `/26`...) | `/64` |
| Broadcast | Yes | **No** (multicast instead) |
| Address resolution | ARP | **NDP** |
| NAT | Very common | Not needed |
| Auto-configuration | DHCP (or manual) | **SLAAC**, DHCPv6, or manual |
| Fragmentation | Routers can fragment | Only the sending device fragments |
| Private addresses | `10.x`, `172.16-31.x`, `192.168.x` | `fd00::/8` (unique local) |
| Loopback | `127.0.0.1` | `::1` |

---

## 9. IPv4 and IPv6 Living Together

IPv6 is **not backward compatible** with IPv4: an IPv4-only device cannot talk directly to an IPv6-only device. So the world is moving over slowly, and both run side by side.

| Transition method | Idea |
|-------------------|------|
| **Dual stack** | Device or network runs **both** IPv4 and IPv6 at the same time (the most common approach) |
| **Tunneling** | IPv6 packets are wrapped inside IPv4 packets to cross an IPv4-only network |
| **Translation (NAT64/DNS64)** | A gateway translates between IPv6-only clients and IPv4-only servers |

Adoption is growing but uneven between countries and networks, so check current statistics if you need numbers.

---

## 10. Hands-On Lab

See your own IPv6 addresses.

1. **Show your addresses**
   ```
   ipconfig            # Windows, look for "IPv6 Address" and "Link-local IPv6 Address"
   ip -6 addr          # Linux
   ifconfig            # macOS, look for lines starting with "inet6"
   ```
2. **Spot the types.** Find the `fe80::` (link-local) address and any `2xxx:` (global) address. Which one is temporary (privacy)?
3. **Ping yourself**
   ```
   ping ::1                  # Windows / macOS: ping -6 ::1 also works
   ping -6 localhost         # forces IPv6
   ```
4. **Ping an IPv6 host** (works only if your network has IPv6)
   ```
   ping -6 ipv6.google.com   # Windows / Linux
   ```
5. **See the neighbor table** (the IPv6 "ARP table")
   ```
   ip -6 neigh               # Linux
   netsh interface ipv6 show neighbors    # Windows
   ```
6. **Practice shortening** every address you found, and expand the short ones back to their full 32-digit form.

If you have no global IPv6 address, your ISP or router probably has IPv6 turned off. That is common and not a mistake.

---

## 11. Practice Questions

1. How many bits are in an IPv6 address? How many groups?
2. Shorten `2001:0db8:0000:0000:0000:0000:0000:0001`.
3. Shorten `fe80:0000:0000:0000:0202:b3ff:fe1e:8329`.
4. Expand `2001:db8:0:1::5` to its full form.
5. Why is `2001::db8::1` invalid?
6. Name the type of each: `fe80::1`, `::1`, `2001:db8::1`, `fd12:3456:789a::1`, `ff02::1`.
7. What is the network prefix of `2001:db8:abcd:12::5/64`?
8. How many `/64` subnets fit inside a `/48`?
9. Does IPv6 have a broadcast address? What is used instead?
10. Which protocol replaces ARP in IPv6?
11. Why did IPv6 skip version 5?
12. True or false: with IPv6 and no NAT, every device on your home network is directly reachable from the Internet.

<details>
<summary>Show answers</summary>

1. **128 bits**, in **8 groups** of 16 bits (4 hex digits each).
2. `2001:db8::1`
3. `fe80::202:b3ff:fe1e:8329`
4. `2001:0db8:0000:0001:0000:0000:0000:0005` (visible groups: 5, so `::` stands for 3 zero groups)
5. It has **two** `::`, which makes the address ambiguous. Only one is allowed.
6. `fe80::1` link-local; `::1` loopback; `2001:db8::1` global unicast (documentation range); `fd12:3456:789a::1` unique local (private); `ff02::1` multicast (all nodes on the link).
7. `2001:db8:abcd:12::/64` (first 4 groups kept, the rest zeroed).
8. 2^16 = **65,536**.
9. **No.** It uses **multicast** (for example `ff02::1` for all nodes).
10. **NDP (Neighbor Discovery Protocol).**
11. Version 5 was already used for an experimental streaming protocol (ST) that never became a standard.
12. **False.** Devices can have public addresses, but the router's **firewall** still blocks unsolicited incoming connections by default.

</details>

---

## 12. Quick Cheat Sheet

| Concept | Remember |
|---------|----------|
| Size | 128 bits = 8 groups x 16 bits = 32 hex digits |
| Notation | Colon-separated hex, lowercase recommended |
| Hex digit | 1 digit = 4 bits |
| Shorten rule 1 | Drop **leading** zeros in each group |
| Shorten rule 2 | Replace **one** run of all-zero groups with `::` (only once) |
| Prefix | Network part, written `/n` like CIDR |
| Standard LAN | **/64** (64-bit prefix + 64-bit interface ID) |
| ISP gives | Often `/48` (business) or `/56` (home) |
| Global unicast | `2000::/3` (public) |
| Link-local | `fe80::/10` (automatic, local link only) |
| Unique local | `fd00::/8` (private) |
| Multicast | `ff00::/8` |
| Loopback / unspecified | `::1` / `::` |
| Docs / examples | `2001:db8::/32` |
| Broadcast | **None** (multicast instead) |
| ARP replaced by | **NDP** |
| Auto config | **SLAAC** (or DHCPv6) |

### Remember in one sentence

> **IPv6 was created because IPv4's 4.3 billion addresses ran out: it uses 128-bit addresses written as 8 groups of hex separated by colons, shortened by dropping leading zeros and using `::` once.**

---

*Notes by [your name]. Feel free to open an issue or PR if you spot a mistake.*
