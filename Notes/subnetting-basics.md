# Subnet Masks, CIDR, and Calculating Subnets Fast

> Beginner-friendly notes. Subnetting is a **skill**, not a theory topic: read the method once, then do the drills in section 9 until you stop having to think.
>
> Prerequisites: [Networking Basics](networking-basics.md) and [IPv4 Basics](ipv4-basics.md) (binary and the 128 64 32 16 8 4 2 1 row).

## Table of Contents

1. [What Is a Subnet Mask?](#1-what-is-a-subnet-mask)
2. [CIDR Notation](#2-cidr-notation)
3. [The 8 Mask Values You Must Memorize](#3-the-8-mask-values-you-must-memorize)
4. [Why Subnet at All?](#4-why-subnet-at-all)
5. [The Fast Method (Block Size)](#5-the-fast-method-block-size)
6. [Worked Examples](#6-worked-examples)
7. [Splitting a Network Into Subnets](#7-splitting-a-network-into-subnets)
8. [Master Cheat Table](#8-master-cheat-table)
9. [Practice Drills (Make It Automatic)](#9-practice-drills-make-it-automatic)
10. [Common Mistakes](#10-common-mistakes)

---

## 1. What Is a Subnet Mask?

An IP address has a **network part** and a **host part**. The **subnet mask** tells a device where the split is.

- A **1** in the mask = this bit is part of the **network**
- A **0** in the mask = this bit is part of the **host**

The 1s always come first, then the 0s. Never mixed.

```
IP address:   192.168.1.10   = 11000000.10101000.00000001.00001010
Mask:         255.255.255.0  = 11111111.11111111.11111111.00000000
                               \______ network ______/ \_ host _/
```

### What a device does with the mask

It does a bitwise **AND** between the IP address and the mask to find its **network address**. (1 AND 1 = 1, anything else = 0.)

```
IP:      192.168.1.130   = 11000000.10101000.00000001.10000010
Mask:    255.255.255.192 = 11111111.11111111.11111111.11000000
                           ---------------------------------- AND
Network: 192.168.1.128   = 11000000.10101000.00000001.10000000
```

This is how a device decides: "Is the destination in **my** network (send directly) or **another** one (send to the router)?"

---

## 2. CIDR Notation

**CIDR** (Classless Inter-Domain Routing) writes the mask as a **slash and a number**: the count of 1s in the mask.

```
255.255.255.0    ->  24 ones  ->  /24
255.255.255.192  ->  26 ones  ->  /26
255.255.0.0      ->  16 ones  ->  /16
```

So `192.168.1.10/24` means "this IP, with a mask of 24 network bits."

### Why CIDR replaced classes

Classes forced masks of only /8, /16, or /24. CIDR allows **any** length (/20, /27, /30...), so you can give a network exactly the size it needs instead of wasting addresses.

### CIDR number to mask (fast)

1. Divide the number by 8 to see how many **full octets** of 255 you get.
2. The **remainder** is the number of 1s in the next octet. Look it up in the table in section 3.
3. Everything after is 0.

Example `/20`: 20 = 8 + 8 + **4** -> `255.255.` + (4 bits = 240) + `.0` = **255.255.240.0**

Example `/27`: 27 = 8 + 8 + 8 + **3** -> `255.255.255.` + (3 bits = 224) = **255.255.255.224**

### Mask to CIDR number

Count the 1s. Each `255` = 8. Add the bits from the one partial octet.

`255.255.248.0` -> 8 + 8 + (248 = 5 bits) = **/21**

---

## 3. The 8 Mask Values You Must Memorize

Only **eight** numbers ever appear in a mask octet:

| 1s in the octet | Binary | Decimal | Block size (256 - decimal) |
|-----------------|--------|---------|----------------------------|
| 1 | 10000000 | **128** | 128 |
| 2 | 11000000 | **192** | 64 |
| 3 | 11100000 | **224** | 32 |
| 4 | 11110000 | **240** | 16 |
| 5 | 11111000 | **248** | 8 |
| 6 | 11111100 | **252** | 4 |
| 7 | 11111110 | **254** | 2 |
| 8 | 11111111 | **255** | 1 |

Notice the pattern: the **block size halves** each row: 128, 64, 32, 16, 8, 4, 2, 1. The mask value is always `256 - block size`.

If you memorize only one table in this whole document, memorize this one.

---

## 4. Why Subnet at All?

**Subnetting** = cutting one big network into several smaller ones.

Reasons:

- **Less broadcast traffic:** a broadcast only reaches its own subnet, so the network is less noisy.
- **Security:** you can separate departments (Finance, Guests, Servers) and control traffic between them with the router/firewall.
- **Better address use:** give each group only what it needs.
- **Easier management:** problems stay contained.

The cost: every subnet loses 2 addresses (network + broadcast).

### The two formulas

Let **h** = number of host bits (the 0s in the mask), and **b** = number of bits you borrowed from the host part to make subnets.

```
Usable hosts per subnet  =  2^h - 2
Number of subnets        =  2^b
```

(You must always subtract 2 for hosts: one network address, one broadcast address.)

Powers of 2 to know by heart:

| 2^1 | 2^2 | 2^3 | 2^4 | 2^5 | 2^6 | 2^7 | 2^8 | 2^9 | 2^10 |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|------|
| 2 | 4 | 8 | 16 | 32 | 64 | 128 | 256 | 512 | 1024 |

---

## 5. The Fast Method (Block Size)

No binary needed. For any `IP/CIDR`, follow these steps.

### Step 1: Find the "interesting octet"

It is the octet where the mask is **not 255 and not 0**.

| CIDR | Interesting octet |
|------|-------------------|
| /9 - /15 | 2nd |
| /17 - /23 | 3rd |
| /25 - /30 | 4th |
| /8, /16, /24 | none (mask is a clean boundary) |

### Step 2: Find the block size

`Block size = 256 - (mask value in that octet)`

### Step 3: List the multiples

Subnets start at multiples of the block size: `0, B, 2B, 3B...` up to 256.

### Step 4: Find your IP's block

Find the **largest multiple that is <= your IP's number** in the interesting octet.

### Step 5: Read off the answers

| Answer | How to get it |
|--------|---------------|
| **Network address** | The multiple you found (octets before it stay the same, octets after it are 0) |
| **Broadcast address** | (Next multiple - 1); octets after it are 255 |
| **First usable host** | Network address + 1 |
| **Last usable host** | Broadcast address - 1 |
| **Usable hosts** | 2^h - 2 |

---

## 6. Worked Examples

### Example 1: `192.168.1.130/26`

1. Mask: /26 = `255.255.255.192`. Interesting octet = **4th**.
2. Block size = 256 - 192 = **64**.
3. Multiples: 0, 64, 128, 192.
4. 130 sits in the **128** block (128 <= 130 < 192).
5. Answers:

| | |
|---|---|
| Network | 192.168.1.**128** |
| First host | 192.168.1.129 |
| Last host | 192.168.1.190 |
| Broadcast | 192.168.1.**191** (next block 192, minus 1) |
| Usable hosts | 2^6 - 2 = **62** |

### Example 2: `172.16.35.77/20`

1. /20 = `255.255.240.0`. Interesting octet = **3rd** (mask 240).
2. Block size = 256 - 240 = **16**.
3. Multiples in the 3rd octet: 0, 16, 32, 48...
4. The 3rd octet of our IP is 35, which sits in the **32** block (32 <= 35 < 48).
5. Answers:

| | |
|---|---|
| Network | 172.16.**32**.0 |
| First host | 172.16.32.1 |
| Last host | 172.16.**47.254** |
| Broadcast | 172.16.**47.255** (next block 48, minus 1; the octets after it become 255) |
| Usable hosts | 2^12 - 2 = **4,094** |

### Example 3: `10.1.1.200/27`

1. /27 = `255.255.255.224`. Interesting octet = **4th**.
2. Block size = 256 - 224 = **32**.
3. Multiples: 0, 32, 64, 96, 128, 160, 192, 224.
4. 200 sits in the **192** block.
5. Network = **10.1.1.192**, broadcast = **10.1.1.223**, hosts = 10.1.1.193 to 10.1.1.222, usable = 2^5 - 2 = **30**.

### Example 4: `192.168.10.5/30` (a router-to-router link)

1. /30 = `255.255.255.252`, block size = **4**.
2. 5 sits in the **4** block (4-7).
3. Network = **.4**, broadcast = **.7**, usable hosts = **.5** and **.6**. That is 2^2 - 2 = **2** hosts, exactly enough for two routers.

---

## 7. Splitting a Network Into Subnets

### Case A: "I need N subnets"

Example: split `192.168.1.0/24` into **4 subnets**.

1. Need 4 subnets -> 2^b >= 4 -> **b = 2** bits to borrow.
2. New mask: /24 + 2 = **/26** (`255.255.255.192`).
3. Block size = 256 - 192 = **64**.
4. Result:

| Subnet | Network | Usable range | Broadcast |
|--------|---------|--------------|-----------|
| 1 | 192.168.1.0/26 | .1 - .62 | .63 |
| 2 | 192.168.1.64/26 | .65 - .126 | .127 |
| 3 | 192.168.1.128/26 | .129 - .190 | .191 |
| 4 | 192.168.1.192/26 | .193 - .254 | .255 |

Each subnet has 62 usable hosts.

### Case B: "I need H hosts per subnet"

Example: a department needs **50 hosts**.

1. Find the smallest h where 2^h - 2 >= 50.
   - 2^5 - 2 = 30 (too small)
   - 2^6 - 2 = 62 (enough) -> **h = 6**
2. Prefix = 32 - 6 = **/26**.

**Shortcut:** `Prefix = 32 - h`, where h is the smallest number such that 2^h - 2 >= hosts needed.

### Where does the extra space go?

Of the 62 usable addresses, 50 are used and 12 are spare. That is fine; some room to grow is good. (Using different mask sizes for different subnets in one network is called **VLSM**, a natural next topic.)

---

## 8. Master Cheat Table

| CIDR | Subnet mask | Block size (interesting octet) | Usable hosts |
|------|-------------|--------------------------------|--------------|
| /8 | 255.0.0.0 | 1 (1st octet) | 16,777,214 |
| /9 | 255.128.0.0 | 128 | 8,388,606 |
| /10 | 255.192.0.0 | 64 | 4,194,302 |
| /11 | 255.224.0.0 | 32 | 2,097,150 |
| /12 | 255.240.0.0 | 16 | 1,048,574 |
| /13 | 255.248.0.0 | 8 | 524,286 |
| /14 | 255.252.0.0 | 4 | 262,142 |
| /15 | 255.254.0.0 | 2 | 131,070 |
| /16 | 255.255.0.0 | 1 (2nd octet) | 65,534 |
| /17 | 255.255.128.0 | 128 | 32,766 |
| /18 | 255.255.192.0 | 64 | 16,382 |
| /19 | 255.255.224.0 | 32 | 8,190 |
| /20 | 255.255.240.0 | 16 | 4,094 |
| /21 | 255.255.248.0 | 8 | 2,046 |
| /22 | 255.255.252.0 | 4 | 1,022 |
| /23 | 255.255.254.0 | 2 | 510 |
| /24 | 255.255.255.0 | 1 (3rd octet) | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |

Special cases: **/31** (255.255.255.254) is used for point-to-point links and has 2 usable addresses with no network/broadcast; **/32** (255.255.255.255) is a single host.

---

## 9. Practice Drills (Make It Automatic)

Speed comes from repetition. Cover the answers, write your own, then check. **Do each drill until you can finish it without looking anything up.**

### Suggested routine

| Day | Do this |
|-----|---------|
| 1-2 | Memorize section 3 (the 8 values) and the powers of 2. Do Drills A and B. |
| 3-4 | Do Drills C and D. Say the block size out loud before each problem. |
| 5-6 | Do Drills E and F. Time yourself. |
| 7 | Make 10 random problems for yourself (any IP, any /prefix) and solve them. Goal: **under 30 seconds** each. |

Ask yourself the same 4 questions every time:
1. What is the mask? 2. What is the block size? 3. Which block is the IP in? 4. What are the network and broadcast addresses?

### Drill A: CIDR to mask

1. /26  2. /20  3. /29  4. /18  5. /12

<details>
<summary>Answers</summary>

1. 255.255.255.192
2. 255.255.240.0
3. 255.255.255.248
4. 255.255.192.0
5. 255.240.0.0

</details>

### Drill B: Mask to CIDR

1. 255.255.255.224  2. 255.255.254.0  3. 255.255.252.0  4. 255.248.0.0  5. 255.255.255.252

<details>
<summary>Answers</summary>

1. /27
2. /23
3. /22
4. /13
5. /30

</details>

### Drill C: Usable hosts

1. /28  2. /25  3. /22  4. /16  5. /30

<details>
<summary>Answers</summary>

1. 14 (2^4 - 2)
2. 126 (2^7 - 2)
3. 1,022 (2^10 - 2)
4. 65,534 (2^16 - 2)
5. 2 (2^2 - 2)

</details>

### Drill D: Full subnet calculation

Find the **network address, broadcast address, usable range, and number of usable hosts**.

1. `192.168.5.77/26`
2. `10.20.30.40/28`
3. `172.16.200.9/21`
4. `192.168.1.99/25`
5. `10.0.0.250/30`
6. `172.20.100.5/18`

<details>
<summary>Answers</summary>

| # | Network | Broadcast | Usable range | Hosts |
|---|---------|-----------|--------------|-------|
| 1 | 192.168.5.64 | 192.168.5.127 | .65 - .126 | 62 |
| 2 | 10.20.30.32 | 10.20.30.47 | .33 - .46 | 14 |
| 3 | 172.16.200.0 | 172.16.207.255 | 172.16.200.1 - 172.16.207.254 | 2,046 |
| 4 | 192.168.1.0 | 192.168.1.127 | .1 - .126 | 126 |
| 5 | 10.0.0.248 | 10.0.0.251 | .249 - .250 | 2 |
| 6 | 172.20.64.0 | 172.20.127.255 | 172.20.64.1 - 172.20.127.254 | 16,382 |

Quick reasoning for a few:
- #1: /26, block 64, 77 is in the 64 block.
- #3: /21, block 8 in the 3rd octet, 200 is exactly a multiple of 8 (8 x 25), so the network starts at 200 and ends at 207.
- #6: /18, block 64 in the 3rd octet, 100 is in the 64 block (64-127).

</details>

### Drill E: Same subnet or not?

Can these two hosts talk directly, without a router?

1. `192.168.1.60/26` and `192.168.1.70/26`
2. `10.0.5.20/24` and `10.0.5.200/24`
3. `172.16.17.5/20` and `172.16.30.9/20`

<details>
<summary>Answers</summary>

1. **No.** /26 blocks are 0-63 and 64-127. 60 is in the first, 70 in the second.
2. **Yes.** Both are in 10.0.5.0/24.
3. **Yes.** /20 has 3rd-octet blocks of 16. 17 and 30 are both in the 16-31 block (172.16.16.0/20).

</details>

### Drill F: Design questions

1. A team needs **100 hosts**. What prefix?
2. A team needs **20 hosts**. What prefix?
3. A link between two routers needs **2 addresses**. What prefix?
4. Split `192.168.10.0/24` into **8 subnets**. New mask, block size, first 3 subnets?

<details>
<summary>Answers</summary>

1. 2^7 - 2 = 126 >= 100, so h = 7, **/25**.
2. 2^5 - 2 = 30 >= 20, so h = 5, **/27**.
3. **/30** (2 usable hosts).
4. 8 subnets needs 3 bits, so **/27** (255.255.255.224), block size **32**.
   Subnets: `192.168.10.0/27`, `192.168.10.32/27`, `192.168.10.64/27`, ... up to `192.168.10.224/27`.

</details>

---

## 10. Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting to subtract 2 from hosts | Always `2^h - 2` (except /31 and /32 special cases) |
| Using the wrong block size | Block size = `256 - mask value`, never the CIDR number |
| Broadcast off by one | Broadcast = **next block start - 1** |
| Forgetting octets after the interesting one | After the network octet they are `0`; in the broadcast they are `255` |
| Mixing up "subnets" and "hosts" formulas | Subnets use **borrowed** bits (b), hosts use **remaining** bits (h) |
| Treating mask octets as random numbers | There are only 8 valid values: 128, 192, 224, 240, 248, 252, 254, 255 |

---

## Quick Summary

| Concept | Remember |
|---------|----------|
| Subnet mask | 1s = network, 0s = host |
| CIDR | `/n` = number of 1s in the mask |
| Valid mask octets | 128, 192, 224, 240, 248, 252, 254, 255 |
| Block size | 256 - mask value |
| Hosts | 2^h - 2 |
| Subnets | 2^b |
| Network address | Start of the block |
| Broadcast address | Next block - 1 |

### Remember in one sentence

> The subnet mask draws the line between network and host; to subnet fast, **find the block size (256 minus the mask value), find which block your IP falls into, and the broadcast is always one less than the start of the next block**.

---

*Notes by [your name]. Feel free to open an issue or PR if you spot a mistake.*
