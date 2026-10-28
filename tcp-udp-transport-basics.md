# Reliable vs Unreliable Transport: TCP, UDP, and the Three-Way Handshake

> Beginner-friendly notes. This zooms into **Layer 4 (Transport)** from [The OSI Model](osi-model-basics.md) and [OSI vs TCP/IP](osi-vs-tcpip-mapping.md). Earlier notes: [Networking Basics](networking-basics.md), [IPv4](ipv4-basics.md), [Subnetting](subnetting-basics.md), [ARP and MAC](arp-and-mac-basics.md), [IPv6](ipv6-basics.md).

## Table of Contents

1. [The Problem: The Network Underneath Is Unreliable](#1-the-problem-the-network-underneath-is-unreliable)
2. [What "Reliable" Means](#2-what-reliable-means)
3. [UDP: Unreliable Transport](#3-udp-unreliable-transport)
4. [TCP: Reliable Transport](#4-tcp-reliable-transport)
5. [The Three-Way Handshake](#5-the-three-way-handshake)
6. [After the Handshake: Sequence Numbers and ACKs](#6-after-the-handshake-sequence-numbers-and-acks)
7. [Flow Control and Congestion Control](#7-flow-control-and-congestion-control)
8. [Closing a Connection](#8-closing-a-connection)
9. [TCP vs UDP Comparison](#9-tcp-vs-udp-comparison)
10. [When Each Is Used](#10-when-each-is-used)
11. [Hands-On Lab](#11-hands-on-lab)
12. [Practice Questions](#12-practice-questions)
13. [Quick Cheat Sheet](#13-quick-cheat-sheet)

---

## 1. The Problem: The Network Underneath Is Unreliable

Layer 3 (**IP**) is **best effort**. It tries to deliver each packet, but promises nothing:

| What can go wrong | Why |
|-------------------|-----|
| **Packet lost** | A router's queue is full (congestion), a link fails, the TTL hits 0 |
| **Packet corrupted** | Electrical interference or a faulty cable (usually caught and dropped at Layer 2) |
| **Packets out of order** | Different packets take different routes |
| **Packet duplicated** | Retransmissions or network quirks |
| **Packet delayed** | Long routes, congested links |

IP does not notice or fix any of this. So the question is left to **Layer 4**:

> **Should the transport layer repair these problems, or just pass the raw behavior up to the application?**

The two answers are the two famous transport protocols:

| | Answer | Protocol |
|---|--------|----------|
| "Yes, fix everything." | **Reliable** transport | **TCP** |
| "No, just send it fast." | **Unreliable** transport | **UDP** |

---

## 2. What "Reliable" Means

A reliable transport protocol guarantees that the data the sender gives to it arrives:

1. **Completely**, with nothing lost
2. **Correctly**, with nothing damaged
3. **In order**, the same sequence it was sent
4. **Without duplicates**

If it cannot do that (for example, the cable is cut), it **reports an error** instead of silently pretending.

### Analogy

| | Reliable (TCP) | Unreliable (UDP) |
|---|----------------|------------------|
| Real life | **Registered mail with tracking and a signature** | **Dropping flyers in letterboxes** or sending a **postcard** |
| Behavior | Confirms delivery, resends if lost | Sends and moves on |
| Cost | Slower, more overhead | Fast, light |

### Important: "unreliable" does not mean "bad"

It only means **no guarantees from the transport layer**. Many important applications want exactly that, and add their own rules where needed (section 10).

---

## 3. UDP: Unreliable Transport

**UDP = User Datagram Protocol.** It is the minimal transport protocol: it adds **ports** (so data reaches the right program) and an optional error check, and that is about it.

### What UDP does

- Adds **source and destination ports** (so it can find the right application)
- Adds a **checksum** to detect corrupted data (a bad datagram is dropped; it is not repaired)
- Sends each message as an independent **datagram**

### What UDP does NOT do

| Not provided | Meaning |
|--------------|---------|
| No connection setup | No handshake. It just sends. |
| No acknowledgements | The sender never learns whether it arrived |
| No retransmission | Lost data stays lost (unless the app handles it) |
| No ordering | Datagrams can arrive in a different order |
| No flow control | A fast sender can overwhelm a slow receiver |
| No congestion control | It does not slow down when the network is busy |

### UDP header: only 8 bytes

```
 0               16              32 bits
 +---------------+---------------+
 | Source port   | Dest. port    |
 +---------------+---------------+
 | Length        | Checksum      |
 +---------------+---------------+
 |          Data ...             |
```

(In IPv4 the UDP checksum is optional; in IPv6 it is mandatory.)

### Other properties

- **Message-oriented:** each send is one datagram, and boundaries are preserved. If you send 3 datagrams, the receiver gets 3 separate datagrams (if they arrive).
- **Supports broadcast and multicast:** one sender to many receivers (TCP can only do one-to-one).
- **Low latency:** nothing waits for confirmation.

**Analogy:** shouting a message across a room. It is fast and needs no setup, but you do not know who heard it.

---

## 4. TCP: Reliable Transport

**TCP = Transmission Control Protocol.** It builds a dependable, ordered **stream of bytes** between two programs on top of the unreliable IP network.

### The tools TCP uses to be reliable

| Mechanism | What it does |
|-----------|--------------|
| **Connection setup** (three-way handshake) | Both sides agree to talk and prepare their state |
| **Sequence numbers** | Every byte is numbered, so data can be put back in order and gaps detected |
| **Acknowledgements (ACKs)** | The receiver says which bytes have arrived |
| **Retransmission** | If an ACK does not come in time (a **timeout**), or duplicate ACKs signal a gap, the sender resends |
| **Checksum** | Detects corrupted segments, which are then discarded and resent |
| **Flow control** | The receiver tells the sender how much it can handle (the **window**) |
| **Congestion control** | The sender slows down when the network is overloaded |
| **Connection teardown** | A clean four-step close so no data is cut off |

### Key properties

- **Connection-oriented:** a connection must be opened before data flows.
- **Byte-stream:** TCP sees data as one continuous stream, **with no message boundaries**. If you send "hello" then "world", the receiver might read "helloworld" in one go. The application must mark its own boundaries.
- **Full duplex:** both sides can send at the same time.
- **Point-to-point:** exactly two endpoints (no broadcast or multicast).

### TCP header: 20 bytes minimum

```
 0                 16                32 bits
 +-----------------+-----------------+
 | Source port     | Dest. port      |
 +-----------------+-----------------+
 |          Sequence number          |
 +-----------------------------------+
 |       Acknowledgement number      |
 +------+-------+-------+------------+
 |Offset|Flags  | Window size        |
 +------+-------+--------------------+
 | Checksum        | Urgent pointer  |
 +-----------------+-----------------+
 |      Options (optional) ...       |
```

### TCP flags (the control bits)

| Flag | Name | Meaning |
|------|------|---------|
| **SYN** | Synchronize | "Let's start a connection" (carries the starting sequence number) |
| **ACK** | Acknowledge | "I received everything up to this number" |
| **FIN** | Finish | "I have no more data to send" |
| **RST** | Reset | "Abort this connection immediately" (error, or no service on that port) |
| **PSH** | Push | "Pass this data to the application now" |
| **URG** | Urgent | Urgent data marker (rarely used) |

**Analogy:** a phone call. You dial, the other person picks up and confirms, you talk (and say "sorry, can you repeat that?" if you missed something), and you both say goodbye before hanging up.

---

## 5. The Three-Way Handshake

Before sending any data, TCP opens a connection with **three messages**. This is the **three-way handshake**.

### The goal

Both sides need to:

1. Confirm the other is **alive and reachable**
2. Agree on the **starting sequence number** each side will use (the **ISN**, Initial Sequence Number)
3. Negotiate a few settings (such as the maximum segment size)

### The three steps

The **client** is the side that opens the connection; the **server** is waiting for connections (in the **LISTEN** state).

Example numbers: the client picks ISN `1000`, the server picks ISN `5000`.

```
   Client                                       Server
 (CLOSED)                                      (LISTEN)
    |                                              |
    |  1) SYN      seq=1000                        |
    |--------------------------------------------->|
 (SYN-SENT)                                 (SYN-RECEIVED)
    |                                              |
    |  2) SYN+ACK  seq=5000, ack=1001              |
    |<---------------------------------------------|
    |                                              |
    |  3) ACK      seq=1001, ack=5001              |
    |--------------------------------------------->|
 (ESTABLISHED)                               (ESTABLISHED)
    |                                              |
    |=============== data can flow ================|
```

### Step by step

| Step | Direction | Flags | What it says | Numbers |
|------|-----------|-------|--------------|---------|
| **1. SYN** | Client to Server | SYN | "I want to connect. My sequence numbers will start at **1000**." | seq = 1000 |
| **2. SYN-ACK** | Server to Client | SYN + ACK | "OK. I got your 1000 (so I expect **1001** next). My sequence numbers will start at **5000**." | seq = 5000, ack = 1001 |
| **3. ACK** | Client to Server | ACK | "I got your 5000 (so I expect **5001** next). Let's go." | seq = 1001, ack = 5001 |

### Key rules to remember

- **ACK number = the next byte I expect from you.** That is why it is the other side's sequence number + 1.
- A **SYN counts as one byte** of sequence space (that is why 1000 becomes 1001 even though no data has been sent).
- The first real data byte from the client will be numbered **1001**, and from the server **5001**.
- The ISNs are **random**, not 0 or 1. This keeps old, delayed segments from a previous connection from being confused with the new one, and makes it harder for attackers to guess sequence numbers.

### Why three messages and not two?

Each side must prove **two things**: "I can send" and "I can receive". With three messages, both directions are confirmed:

| Message | What it proves |
|---------|----------------|
| 1. SYN | The client can **send** to the server |
| 2. SYN-ACK | The server can **receive** and **send** (and received the client's SYN) |
| 3. ACK | The client can **receive** (it got the server's SYN) |

With only two messages, the server could never be sure that the client received its SYN-ACK or that the client's SYN was not an old, delayed duplicate.

### Settings agreed in the handshake (TCP options)

| Option | Purpose |
|--------|---------|
| **MSS** (Maximum Segment Size) | The largest chunk of data each side will accept (often 1460 bytes on Ethernet) |
| **Window scaling** | Allows window sizes larger than 65,535 bytes for fast networks |
| **SACK permitted** | Allows selective acknowledgements (smarter loss recovery) |
| **Timestamps** | More accurate timing and protection against old segments |

### What the OS does for you

Your program calls `connect()`. The **operating system** sends the SYN and completes the handshake. On the server side the OS finishes the handshake **before** the program's `accept()` returns. Programs never build the handshake themselves.

### SYN flood (security note)

A **SYN flood** is a classic denial-of-service attack: an attacker sends huge numbers of SYNs and never completes step 3, so the server fills its table of half-open connections. A common defense is **SYN cookies**, where the server encodes the needed information in its ISN instead of keeping state until the final ACK arrives, plus firewalls and rate limiting.

---

## 6. After the Handshake: Sequence Numbers and ACKs

Once the connection is **ESTABLISHED**, data flows and every byte is numbered.

### The rules

- **Sequence number** = the number of the **first byte** in this segment.
- **ACK number** = the number of the **next byte expected**. ACKs are **cumulative**: "ack 1301" means "I have everything up to byte 1300".

### Example: all goes well

Client sends 300 bytes in 3 segments of 100 bytes (starting at 1001):

```
 Client                                   Server
   |  seq=1001  (100 bytes)                  |
   |---------------------------------------->|
   |  seq=1101  (100 bytes)                  |
   |---------------------------------------->|
   |  seq=1201  (100 bytes)                  |
   |---------------------------------------->|
   |                    ack=1301             |
   |<----------------------------------------|   "I have everything up to 1300"
```

(Receivers may send one ACK for several segments; it does not need to be one for each.)

### Example: a segment is lost

```
 Client                                   Server
   |  seq=1001  (100 bytes)                  |
   |---------------------------------------->|
   |                           ack=1101      |
   |<----------------------------------------|
   |  seq=1101  (100 bytes)       X  LOST    |
   |------------------------X                |
   |  seq=1201  (100 bytes)                  |
   |---------------------------------------->|   arrives, but there is a gap
   |                           ack=1101      |   <- duplicate ACK: "still waiting for 1101"
   |<----------------------------------------|
   |  seq=1301  (100 bytes)                  |
   |---------------------------------------->|
   |                           ack=1101      |   <- 2nd duplicate ACK
   |<----------------------------------------|
   |  ... 3rd duplicate ACK ...              |
   |                                         |
   |  seq=1101  (100 bytes)   RETRANSMIT     |   <- "fast retransmit"
   |---------------------------------------->|
   |                           ack=1401      |   <- gap filled, everything up to 1400 received
   |<----------------------------------------|
```

### Two ways TCP detects loss

| Detection | How it works |
|-----------|--------------|
| **Timeout (RTO)** | The sender starts a timer for sent data. If no ACK arrives in time, it resends. The timer adapts to the measured round-trip time. |
| **Duplicate ACKs (fast retransmit)** | After **3 duplicate ACKs**, the sender assumes the segment was lost and resends it right away, without waiting for the timer. |

The receiver also **reorders** segments that arrive out of order, and **discards duplicates**, before giving the data to the application in the correct order.

---

## 7. Flow Control and Congestion Control

These two sound similar but protect against different problems.

| | **Flow control** | **Congestion control** |
|---|------------------|------------------------|
| Protects | The **receiver** from being overwhelmed | The **network** from being overloaded |
| Who decides | The receiver (advertises a **window** in every ACK) | The sender (estimates from losses and delays) |
| Variable | **rwnd**, the receive window | **cwnd**, the congestion window |
| Analogy | "Slow down, my inbox is full." | "The road is jammed, ease off." |

The sender can have at most `min(rwnd, cwnd)` bytes of **unacknowledged data in flight** at once.

### Congestion control in a nutshell

1. **Slow start:** begin with a small window and roughly **double it every round trip** while ACKs come back.
2. **Congestion avoidance:** after a threshold, grow **slowly** (about one segment per round trip).
3. **On loss:** treat it as a sign of congestion, **shrink the window**, and then grow again.

This is why TCP is a "good citizen": many TCP connections naturally share a link fairly. UDP has **no** such mechanism, so a heavy UDP sender can crowd out others unless the application limits itself.

---

## 8. Closing a Connection

A TCP connection is closed in each direction separately, using **FIN**:

```
   Client                                   Server
     |  FIN  (I'm done sending)                |
     |---------------------------------------->|
     |                         ACK             |
     |<----------------------------------------|
     |                  (server may still send data)
     |                         FIN             |
     |<----------------------------------------|
     |  ACK                                    |
     |---------------------------------------->|
  (TIME_WAIT, then CLOSED)                  (CLOSED)
```

- It is usually called a **four-way** close (FIN, ACK, FIN, ACK). Sometimes the middle two combine into one FIN+ACK.
- The side that closes first waits in **TIME_WAIT** for a while (twice the maximum segment lifetime) to make sure the last ACK arrived and to avoid old segments being confused with a new connection.
- **RST** is the abrupt alternative: it kills the connection immediately (for example, when connecting to a port with nothing listening).

### TCP connection states (the ones you will see in `netstat` / `ss`)

| State | Meaning |
|-------|---------|
| **LISTEN** | Server waiting for connections |
| **SYN-SENT** | Client sent SYN, waiting for SYN-ACK |
| **SYN-RECEIVED** | Server got SYN, sent SYN-ACK, waiting for the final ACK |
| **ESTABLISHED** | Connection open, data flowing |
| **FIN-WAIT / CLOSE-WAIT** | The two sides in the middle of closing |
| **TIME-WAIT** | Waiting a short time after closing |
| **CLOSED** | No connection |

---

## 9. TCP vs UDP Comparison

| Feature | **TCP** | **UDP** |
|---------|---------|---------|
| Full name | Transmission Control Protocol | User Datagram Protocol |
| Connection | **Connection-oriented** (handshake) | **Connectionless** |
| Reliability | **Reliable** (ACKs, retransmission) | **Unreliable** (best effort) |
| Ordering | **Guaranteed** | Not guaranteed |
| Duplicate protection | Yes | No |
| Error checking | Checksum + recovery | Checksum only (drops bad data) |
| Flow control | Yes | No |
| Congestion control | Yes | No |
| Data model | **Byte stream** (no message boundaries) | **Datagrams** (boundaries preserved) |
| Header size | **20 bytes** minimum | **8 bytes** |
| Speed / latency | Slower to start, more overhead | Faster, lower latency |
| Broadcast / multicast | **No** (one-to-one only) | **Yes** |
| Data unit | Segment | Datagram |
| IP protocol number | 6 | 17 |

---

## 10. When Each Is Used

### The decision questions

| Question | If YES |
|----------|--------|
| Must **every byte** arrive, correct and in order? (files, web pages, email, logins, money) | **TCP** |
| Is **speed / low delay** more important than perfection? (live voice, video, games) | **UDP** |
| Is it a **tiny request and reply** where a retry is cheap? (DNS lookup) | **UDP** |
| Do you need **broadcast or multicast**? | **UDP** |
| Do you want to **build your own** reliability rules? | **UDP** + application logic |

### Typical applications

| Uses TCP | Why |
|----------|-----|
| **HTTP / HTTPS** (web, traditionally) | A page missing a chunk is broken |
| **Email** (SMTP, IMAP, POP3) | Messages must arrive intact |
| **File transfer** (FTP, SFTP, downloads) | One wrong byte corrupts the file |
| **SSH, remote desktop** | Every keystroke and command must arrive in order |
| **Databases and APIs** | Data integrity matters |

| Uses UDP | Why |
|----------|-----|
| **DNS** (normal lookups) | A tiny request and reply; if lost, just ask again. (DNS switches to TCP for large responses and zone transfers.) |
| **DHCP** | The client has no IP yet, and uses broadcast |
| **Voice and video calls** (VoIP, video conferencing) | A late packet is useless; a tiny glitch is better than a delay |
| **Live streaming** | Same as above |
| **Online gaming** | Fresh position updates matter more than old lost ones |
| **NTP** (time sync), **SNMP** (monitoring) | Small, frequent messages |
| **QUIC / HTTP/3** | Builds reliability on top of UDP (see below) |

### Why retransmitting is bad for live traffic

Suppose a video call loses packet #50.

- **TCP** would **hold back** packets 51, 52, 53... (even though they arrived) until packet #50 is retransmitted and in place, because TCP delivers in order. This delay is called **head-of-line blocking**. By the time #50 arrives, the moment it belonged to has passed, and the conversation stutters.
- **UDP** just delivers 51, 52, 53, and the app skips over the tiny gap (a small glitch).

### "Unreliable" transport with reliable applications

UDP is also a base for protocols that want **custom** reliability:

| Example | What it does |
|---------|--------------|
| **QUIC** (used by HTTP/3) | Reliability, encryption, and multiple independent streams, built on UDP, so one lost packet does not block unrelated streams |
| **Online games / media apps** | Resend only the important messages (like "player died") and ignore stale ones (old positions) |
| **TFTP** | A simple file transfer with its own acknowledgements |

So the choice is not "reliable good, unreliable bad". It is **who decides how to handle loss**: the transport layer (TCP) or the application (UDP).

---

## 11. Hands-On Lab

### Lab 1: Watch the handshake in Wireshark

1. Start a Wireshark capture on your active interface.
2. Open a website in your browser.
3. Stop the capture and use the display filter:
   ```
   tcp.flags.syn == 1
   ```
4. Find a **[SYN]** packet from your IP, then the **[SYN, ACK]** from the server, then the **[ACK]**. Click each and expand **Transmission Control Protocol** to see:
   - the **Sequence number** and **Acknowledgment number** (Wireshark shows "relative" numbers starting at 0 by default; right-click to see raw ones)
   - the **flags**
   - the **options** (MSS, window scale, SACK) in the SYN packets

Only capture on networks you own or have permission to monitor.

### Lab 2: Watch connection states

```
netstat -an          # Windows, macOS, Linux
ss -tan              # Linux (modern)
```

Look for `LISTEN`, `ESTABLISHED`, and `TIME_WAIT`. Open and close a few websites and watch the states change.

### Lab 3: TCP vs UDP in a few lines of Python

**TCP server** (run first):

```python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)   # SOCK_STREAM = TCP
s.bind(("127.0.0.1", 9000))
s.listen()
conn, addr = s.accept()          # the OS has already completed the three-way handshake
print(conn.recv(1024))
```

**TCP client:**

```python
import socket
c = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
c.connect(("127.0.0.1", 9000))   # the three-way handshake happens here
c.send(b"hello")
```

**UDP sender** (no server needed):

```python
import socket
u = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)    # SOCK_DGRAM = UDP
u.sendto(b"hello", ("127.0.0.1", 9001))
print("sent")                    # works even if nobody is listening
```

**Compare:**
- Run the TCP client **without** starting the server first: it fails with "connection refused" (the server's OS answers with an RST).
- Run the UDP sender **without** any receiver: it happily reports "sent", and the data silently disappears. This is "unreliable" in action.

---

## 12. Practice Questions

1. Which layer does TCP work at? What does "reliable" mean for TCP?
2. Is IP reliable or unreliable? Why does that matter for the transport layer?
3. List the three steps of the three-way handshake, with their flags.
4. A client starts with ISN 2000 and the server with ISN 8000. Fill in the numbers:
   (1) SYN seq=?  (2) SYN-ACK seq=?, ack=?  (3) ACK seq=?, ack=?
5. After the handshake in question 4, the client sends 50 bytes of data. What is the sequence number of the first byte? What ACK number does the server send back?
6. Why does the handshake use random initial sequence numbers?
7. Why do we need three messages and not two?
8. How does TCP detect a lost segment? (Give two ways.)
9. What is the difference between flow control and congestion control?
10. Name two applications that use UDP and explain why.
11. What is head-of-line blocking?
12. A program sends 100 bytes and then another 100 bytes over TCP. Is it guaranteed the receiver reads two separate 100-byte chunks?
13. Why can UDP do broadcast and multicast but TCP cannot?
14. A web server's port 443 is closed (nothing listening). What does TCP send back to a client's SYN?

<details>
<summary>Show answers</summary>

1. **Layer 4 (Transport).** Reliable means data arrives **completely, correctly, in order, and without duplicates**, or an error is reported.
2. **Unreliable (best effort).** Packets can be lost, duplicated, reordered, or delayed, and IP does not fix it. So the transport layer must decide whether to fix it (TCP) or not (UDP).
3. **SYN** (client to server), **SYN + ACK** (server to client), **ACK** (client to server).
4. (1) SYN `seq=2000`; (2) SYN-ACK `seq=8000, ack=2001`; (3) ACK `seq=2001, ack=8001`.
5. The first data byte is numbered **2001** (the SYN used up 2000). The server's ACK is **2051** (2001 + 50 bytes: "next byte I expect is 2051").
6. To avoid confusing old delayed segments with a new connection, and to make sequence numbers hard for an attacker to guess.
7. To confirm **both directions** work and **both ISNs** are acknowledged. With two messages the server could never be sure the client received its SYN-ACK or that the first SYN was not an old duplicate.
8. A **timeout** (no ACK within the retransmission timer) and **three duplicate ACKs** (fast retransmit).
9. **Flow control** protects the **receiver** (window advertised by the receiver). **Congestion control** protects the **network** (window decided by the sender based on losses and delays).
10. Examples: **DNS** (tiny request/reply, retry is cheap), **VoIP/video calls** (a late packet is useless, low delay matters), **online games**, **DHCP** (broadcast, no IP yet).
11. When one lost segment **blocks delivery of all later data**, even though it arrived, because TCP must deliver in order. It causes delay in live traffic.
12. **No.** TCP is a **byte stream** with no message boundaries. The receiver might get 200 bytes at once, or 150 then 50. The application must define its own message boundaries.
13. TCP is **connection-oriented and point-to-point**, so it needs exactly two endpoints with agreed state. UDP is connectionless, so one datagram can go to a group.
14. A **RST** (reset), usually shown as "connection refused".

</details>

---

## 13. Quick Cheat Sheet

| Concept | Remember |
|---------|----------|
| IP | Best effort, **unreliable**; transport decides what to fix |
| Reliable | Complete, correct, in order, no duplicates |
| **TCP** | Connection-oriented, reliable, ordered byte stream, flow + congestion control |
| **UDP** | Connectionless, unreliable, fast, 8-byte header, datagrams |
| Handshake | **SYN, SYN-ACK, ACK** |
| ISN | Random starting sequence number (SYN uses up 1) |
| ACK number | **Next byte expected** (sender's seq + data length) |
| Loss detection | **Timeout** or **3 duplicate ACKs** |
| Flow control | Receiver's window (rwnd) |
| Congestion control | Sender's window (cwnd): slow start, then careful growth |
| Close | **FIN, ACK, FIN, ACK**; **RST** = abort |
| Use TCP for | Web, email, files, SSH, databases |
| Use UDP for | DNS lookups, DHCP, voice/video, games, live streaming, QUIC |
| Head-of-line blocking | Lost TCP segment holds up later data |

### Remember in one sentence

> **TCP trades speed for guarantees: it opens a connection with a three-way handshake (SYN, SYN-ACK, ACK) and then numbers, acknowledges, and resends data so it arrives complete and in order; UDP skips all that, which makes it ideal when speed matters more than perfection.**

---

*Notes by [your name]. Feel free to open an issue or PR if you spot a mistake.*
