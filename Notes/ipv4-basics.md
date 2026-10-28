# IPv4: Structure, Binary Basics, Classes, Public vs Private IPs

> Beginner-friendly notes. Read top to bottom: each section builds on the one before.

## Table of Contents

1. [Binary Basics](#1-binary-basics)
2. [IPv4 Structure](#2-ipv4-structure)
3. [Network Part vs Host Part](#3-network-part-vs-host-part)
4. [IP Address Classes](#4-ip-address-classes)
5. [Public vs Private IP Addresses](#5-public-vs-private-ip-addresses)
6. [Special IP Addresses](#6-special-ip-addresses)
7. [Practice Questions](#7-practice-questions)
8. [Quick Cheat Sheet](#8-quick-cheat-sheet)

---

## 1. Binary Basics

Computers only understand **two states**: ON and OFF.
We write them as **1** (on) and **0** (off). This is the **binary** system (base 2).

We normally count in **decimal** (base 10, digits 0-9). Binary uses only digits 0 and 1.

### Each bit has a value

In an 8-bit number, each position is worth double the one to its right:

| Position | 8th | 7th | 6th | 5th | 4th | 3rd | 2nd | 1st |
|----------|-----|-----|-----|-----|-----|-----|-----|-----|
| Value    | 128 | 64  | 32  | 16  | 8   | 4   | 2   | 1   |

Memorize this row. You will use it again and again.

### Binary to decimal

Add up the values where the bit is **1**.

```
Binary:   1  1  0  0  0  0  0  0
Value:   128 64 32 16  8  4  2  1
          ^  ^
Total = 128 + 64 = 192
```

Another one: `00001101`

```
8 + 4 + 1 = 13
```

### Decimal to binary

**Method 1 (subtract):** go left to right through 128, 64, 32... If the number is big enough, write **1** and subtract. Otherwise write **0**.

Convert **172**:

| Value | Fits in remaining? | Bit | Remaining |
|-------|--------------------|-----|-----------|
| 128 | yes (172 >= 128) | 1 | 44 |
| 64 | no | 0 | 44 |
| 32 | yes | 1 | 12 |
| 16 | no | 0 | 12 |
| 8 | yes | 1 | 4 |
| 4 | yes | 1 | 0 |
| 2 | no | 0 | 0 |
| 1 | no | 0 | 0 |

Result: **172 = 10101100**

**Method 2 (divide by 2):** divide by 2 repeatedly, note the remainders, then read them **bottom to top**.

Convert **13**:

```
13 / 2 = 6  remainder 1
 6 / 2 = 3  remainder 0
 3 / 2 = 1  remainder 1
 1 / 2 = 0  remainder 1

Read bottom to top: 1101  (or 00001101 in 8 bits)
```

### Handy reference

| Decimal | Binary (8-bit) |
|---------|----------------|
| 0 | 00000000 |
| 1 | 00000001 |
| 10 | 00001010 |
| 128 | 10000000 |
| 192 | 11000000 |
| 168 | 10101000 |
| 255 | 11111111 |

**Why 255 is the maximum:** eight 1s = 128+64+32+16+8+4+2+1 = **255**. So one 8-bit number ranges from **0 to 255**.

---

## 2. IPv4 Structure

**IPv4** = Internet Protocol **version 4**. An IP address identifies a device on a network.

### The basics

- An IPv4 address is **32 bits** long.
- The 32 bits are split into **4 groups of 8 bits**.
- Each group is called an **octet** (octo = eight).
- We write each octet in decimal and separate them with dots. This is **dotted decimal notation**.

```
Dotted decimal:   192   .   168   .    1    .   10
                   |         |         |         |
Binary:        11000000 . 10101000 . 00000001 . 00001010
                 octet 1   octet 2   octet 3   octet 4
```

### Key facts

| Fact | Value |
|------|-------|
| Total bits | 32 |
| Number of octets | 4 |
| Range of each octet | 0 to 255 |
| Total possible addresses | 2^32 = 4,294,967,296 (about 4.3 billion) |

So `192.168.1.10` is valid, but `192.168.1.300` is **not**, because 300 is bigger than 255.

### Why we ran out of addresses

4.3 billion sounds like a lot, but there are more devices than that today (phones, laptops, TVs, cameras...). That is why:

- **Private IPs + NAT** were introduced (section 5), and
- **IPv6** (128-bit addresses) was created as the long-term fix.

---

## 3. Network Part vs Host Part

Every IP address has **two parts**:

- **Network portion**: which network the device belongs to (like a street name).
- **Host portion**: which device inside that network (like a house number).

```
 192.168.1 . 10
 \_______/   \_/
  network    host
```

All devices in the same network share the **same network part** and have **different host parts**.

**Analogy:** phone numbers. The area code is the network part, the rest is the host part.

How many bits belong to each part is decided by the **subnet mask** (for example `255.255.255.0`). Classes (next section) were the original, fixed way of deciding this.

---

## 4. IP Address Classes

The original design split IPv4 addresses into **5 classes: A, B, C, D, E**.
The class is decided by the **first octet** (more precisely, the first few bits).

### Class overview

| Class | First bits | First octet range | Default mask | Network bits / Host bits | Used for |
|-------|-----------|-------------------|--------------|--------------------------|----------|
| **A** | `0` | 1 - 126 | 255.0.0.0 (/8) | 8 / 24 | Very large networks |
| **B** | `10` | 128 - 191 | 255.255.0.0 (/16) | 16 / 16 | Medium networks |
| **C** | `110` | 192 - 223 | 255.255.255.0 (/24) | 24 / 8 | Small networks |
| **D** | `1110` | 224 - 239 | n/a | n/a | Multicast |
| **E** | `1111` | 240 - 255 | n/a | n/a | Reserved / experimental |

> **Why does Class A say 1-126 and not 0-127?**
> `0.x.x.x` is reserved and `127.x.x.x` is reserved for loopback (see section 6).

### How to spot the class quickly

Just look at the first number:

```
 1 - 126   -> Class A
127        -> Loopback (special)
128 - 191  -> Class B
192 - 223  -> Class C
224 - 239  -> Class D
240 - 255  -> Class E
```

Examples:

| IP address | First octet | Class |
|------------|-------------|-------|
| 10.5.3.1 | 10 | A |
| 172.16.0.5 | 172 | B |
| 192.168.1.10 | 192 | C |
| 224.0.0.5 | 224 | D |

### How many networks and hosts?

| Class | Networks | Hosts per network |
|-------|----------|-------------------|
| A | 126 | 16,777,214 |
| B | 16,384 | 65,534 |
| C | 2,097,152 | 254 |

**Hosts formula:** `2^(host bits) - 2`

Why minus 2? Two addresses in every network are reserved:

- **All host bits 0** = the **network address** (names the network itself)
- **All host bits 1** = the **broadcast address** (talks to everyone in the network)

Example (Class C, `192.168.1.0/24`):

```
Network address:   192.168.1.0     (host bits all 0)
Usable hosts:      192.168.1.1  to  192.168.1.254
Broadcast address: 192.168.1.255   (host bits all 1)

Usable hosts = 2^8 - 2 = 254
```

### Classes today: classful vs classless

Classes wasted a lot of addresses. A company needing 300 addresses was too big for Class C (254) and was given Class B (65,534), wasting tens of thousands.

Modern networks use **CIDR (Classless Inter-Domain Routing)**, where the mask can be any length (like `/22` or `/27`) instead of just /8, /16, /24.

Classes are still taught because they build the foundation for subnetting, and exams still ask about them. Class D (multicast) is still used in practice.

---

## 5. Public vs Private IP Addresses

### Public IP

- **Globally unique**: no two devices on the Internet can have the same public IP.
- **Routable on the Internet**.
- Given by your **ISP** (your router usually gets one).
- Anyone on the Internet can (in principle) reach it.

### Private IP

- Used **only inside a LAN** (home, office, school).
- **Not routable** on the Internet. Internet routers drop them.
- **Free to use**, and anyone can reuse the same ranges in their own network.
- Assigned by your local router (DHCP).

### Private IP ranges (RFC 1918)

| Class | Range | CIDR | Total addresses |
|-------|-------|------|-----------------|
| A | 10.0.0.0 to 10.255.255.255 | 10.0.0.0/8 | 16,777,216 |
| B | 172.16.0.0 to 172.31.255.255 | 172.16.0.0/12 | 1,048,576 |
| C | 192.168.0.0 to 192.168.255.255 | 192.168.0.0/16 | 65,536 |

> **Common trap:** the private Class B range is **172.16 to 172.31**, not all of 172.x.x.x. So `172.20.1.1` is private but `172.32.1.1` is **public**.

Anything not in these ranges (and not special) is generally a **public** address.

### How private devices still reach the Internet: NAT

Private IPs cannot go on the Internet directly. Your router uses **NAT (Network Address Translation)** to swap your private IP for its own public IP.

```
 Phone   192.168.1.5  \
 Laptop  192.168.1.6   >--[ Router + NAT ]-- Public IP: 203.0.113.7 --> Internet
 TV      192.168.1.7  /
   (private IPs)          (translates)         (one shared public IP)
```

Many devices share **one** public IP. That is how NAT slows down the running-out of IPv4 addresses.

### Comparison

| Feature | Public IP | Private IP |
|---------|-----------|------------|
| Reachable from Internet | Yes | No |
| Unique worldwide | Yes | No (reused everywhere) |
| Cost | Paid via ISP | Free |
| Assigned by | ISP / registry | Your local router |
| Example | 8.8.8.8 | 192.168.1.10 |

**Tip:** to check yours, your private IP is shown in network settings (or `ipconfig` on Windows, `ip a` on Linux). Your public IP is what a "what is my IP" website shows.

---

## 6. Special IP Addresses

| Address | Name | Meaning |
|---------|------|---------|
| `127.0.0.1` | Loopback / localhost | "This computer itself". Used to test your own machine. The whole `127.0.0.0/8` block is loopback. |
| `0.0.0.0` | Unspecified | "Any address" / "no address yet". |
| `255.255.255.255` | Limited broadcast | Send to every device on the local network. |
| `169.254.x.x` | APIPA (link-local) | A device gives itself this when it **fails to get an IP from DHCP**. If you see this, something is wrong with DHCP or the connection. |
| `x.x.x.0` (host bits 0) | Network address | Identifies the network, not a device. |
| `x.x.x.255` (host bits 1) | Broadcast address | Reaches all devices in that network. |

---

## 7. Practice Questions

Try these first, then check the answers.

1. Convert `00001010` to decimal.
2. Convert `200` to binary.
3. What class is `150.10.5.1`?
4. How many usable hosts are in a Class C network?
5. Is `172.31.4.9` public or private?
6. Is `172.32.4.9` public or private?
7. Is `192.168.1.300` a valid IPv4 address?
8. Your PC shows `169.254.22.8`. What does that probably mean?

<details>
<summary>Show answers</summary>

1. `8 + 2 = 10`
2. `200 = 128 + 64 + 8` -> **11001000**
3. First octet 150 is in 128-191 -> **Class B**
4. `2^8 - 2 =` **254**
5. **Private** (inside 172.16 - 172.31)
6. **Public** (just outside the private range)
7. **No**: an octet can only go from 0 to 255
8. It could not get an address from DHCP, so it assigned itself an APIPA address. Check the router/cable/Wi-Fi.

</details>

---

## 8. Quick Cheat Sheet

| Concept | Remember |
|---------|----------|
| IPv4 size | 32 bits = 4 octets x 8 bits |
| Octet range | 0 - 255 |
| Notation | Dotted decimal, e.g. `192.168.1.10` |
| Bit values | 128 64 32 16 8 4 2 1 |
| Class A | 1-126, mask /8 |
| Class B | 128-191, mask /16 |
| Class C | 192-223, mask /24 |
| Class D | 224-239, multicast |
| Class E | 240-255, reserved |
| Usable hosts | 2^(host bits) - 2 |
| Private ranges | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` |
| Loopback | `127.0.0.1` |
| DHCP failed | `169.254.x.x` |

### Remember in one sentence

> An IPv4 address is **32 bits written as 4 numbers (0-255)**; it has a **network part and a host part**; and **private addresses stay inside your LAN** while **public addresses are unique on the Internet**.

---

*Notes by [your name]. Feel free to open an issue or PR if you spot a mistake.*
