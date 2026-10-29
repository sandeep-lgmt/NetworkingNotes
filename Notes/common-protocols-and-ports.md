# Common Application Protocols and Port Numbers: HTTP, HTTPS, DNS, DHCP, FTP, SSH, Telnet

> Beginner-friendly notes. These are all **Layer 7 (Application)** protocols that ride on **TCP or UDP** ports. Read [TCP vs UDP](tcp-udp-transport-basics.md) and [The OSI Model](osi-model-basics.md) first. Earlier notes: [Networking Basics](networking-basics.md), [IPv4](ipv4-basics.md), [Subnetting](subnetting-basics.md), [ARP and MAC](arp-and-mac-basics.md), [IPv6](ipv6-basics.md), [OSI vs TCP/IP](osi-vs-tcpip-mapping.md).

## Table of Contents

1. [Ports in 60 Seconds](#1-ports-in-60-seconds)
2. [Master Table](#2-master-table)
3. [HTTP](#3-http-port-80)
4. [HTTPS](#4-https-port-443)
5. [DNS](#5-dns-port-53)
6. [DHCP](#6-dhcp-ports-67-and-68)
7. [FTP](#7-ftp-ports-21-and-20)
8. [SSH](#8-ssh-port-22)
9. [Telnet](#9-telnet-port-23)
10. [Insecure vs Secure Pairs](#10-insecure-vs-secure-pairs)
11. [More Ports Worth Knowing](#11-more-ports-worth-knowing)
12. [Putting It Together: A Normal Day on the Network](#12-putting-it-together-a-normal-day-on-the-network)
13. [Hands-On Lab](#13-hands-on-lab)
14. [Practice Questions](#14-practice-questions)
15. [Quick Cheat Sheet](#15-quick-cheat-sheet)

---

## 1. Ports in 60 Seconds

An **IP address** finds the **device**. A **port number** finds the **program (service)** on that device.

```
203.0.113.50 : 443     -> "the HTTPS service on that server"
192.168.1.10 : 51514   -> "the browser tab on my PC" (a random port picked by my PC)
```

- A port is a **16-bit number**: `0` to `65535`.
- A server **listens** on a fixed, well-known port. A client uses a **random temporary (ephemeral) port** for its side of the conversation.
- The **transport protocol** matters too: TCP port 53 and UDP port 53 are different "doors".

| Range | Name | Use |
|-------|------|-----|
| **0 - 1023** | **Well-known ports** | Standard services (the ones in these notes) |
| **1024 - 49151** | Registered ports | Specific applications (databases, games...) |
| **49152 - 65535** | Dynamic / ephemeral | Temporary client-side ports |

> **Ports are conventions, not laws.** You *can* run a web server on port 8080 or SSH on port 2222. The well-known numbers are simply the defaults everyone agrees on, so clients know where to knock.

---

## 2. Master Table

| Protocol | Port(s) | Transport | Encrypted? | Main use |
|----------|---------|-----------|------------|----------|
| **HTTP** | **80** | TCP | No | Web pages and APIs (plain) |
| **HTTPS** | **443** | TCP (and UDP for HTTP/3) | **Yes** (TLS) | Secure web pages and APIs |
| **DNS** | **53** | **UDP** (and TCP) | No (unless DoT/DoH) | Names to IP addresses |
| **DHCP** | **67** (server), **68** (client) | **UDP** | No | Automatic IP configuration |
| **FTP** | **21** (control), 20 (data, active mode) | TCP | No | File transfer |
| **SSH** | **22** | TCP | **Yes** | Secure remote login, file transfer, tunnels |
| **Telnet** | **23** | TCP | No | Old remote terminal (insecure) |

**Memory trick:** `20/21` FTP, `22` SSH, `23` Telnet, they are three numbers in a row. Then `53` DNS, `67/68` DHCP, `80` HTTP, `443` HTTPS.

---

## 3. HTTP (Port 80)

**HTTP = HyperText Transfer Protocol.** The protocol of the web.

| | |
|---|---|
| **Port** | **TCP 80** |
| **Model** | Request / response: the client asks, the server answers |
| **Encrypted** | **No**, everything travels as readable text |

### What it looks like

```
GET /index.html HTTP/1.1
Host: www.example.com

HTTP/1.1 200 OK
Content-Type: text/html

<html> ... </html>
```

### Key concepts

| Concept | Meaning |
|---------|---------|
| **Methods** | `GET` (fetch), `POST` (send data), `PUT` (replace), `DELETE` (remove) |
| **Status codes** | `1xx` info, **`2xx` success** (200 OK), **`3xx` redirect** (301), **`4xx` client error** (404 Not Found), **`5xx` server error** (500) |
| **Stateless** | Each request stands alone; sites use **cookies** or tokens to remember you |
| **Versions** | HTTP/1.1 and HTTP/2 run over TCP; **HTTP/3 runs over QUIC on UDP** |

### Use cases

- Browsing websites (today most of it is redirected to HTTPS)
- REST **APIs** between programs and services
- Internal tools, testing, and local development (`http://localhost:8080`)
- Router and device admin pages on a local network

### Security

Anyone who can see the traffic (on shared Wi-Fi, or a compromised router) can **read and modify** it: passwords, cookies, page contents. That is why HTTP is replaced by HTTPS for anything sensitive, and browsers warn "Not secure".

---

## 4. HTTPS (Port 443)

**HTTPS = HTTP Secure.** It is **HTTP carried inside a TLS-encrypted connection**. It is the same protocol with a protective tunnel around it.

| | |
|---|---|
| **Port** | **TCP 443** (and **UDP 443** for HTTP/3 over QUIC) |
| **Encrypted** | **Yes**, using **TLS** (Transport Layer Security, the successor to SSL) |

### What TLS gives you

| Guarantee | Meaning |
|-----------|---------|
| **Confidentiality** | Nobody in the middle can read the data |
| **Integrity** | Nobody can change the data without being detected |
| **Authentication** | You are really talking to `example.com`, not an impostor |

### How it connects (simplified)

```
Client                                         Server
  |-- TCP three-way handshake (port 443) -------->|
  |-- TLS: "Hello, I support these ciphers" ----->|
  |<- TLS: "Here is my certificate + key info" ---|
  |   (client checks the certificate)             |
  |-- keys agreed ------------------------------->|
  |==== everything from here is encrypted =======|
  |-- GET /index.html (encrypted) --------------->|
```

### Certificates

A **certificate** proves the server's identity. It is **signed by a Certificate Authority (CA)** that your browser trusts. The browser checks:

1. Is it signed by a trusted CA?
2. Does the name match the site (`example.com`)?
3. Has it expired?

If any check fails, you get a certificate warning. The padlock icon means the connection is encrypted and the identity checked. It does **not** mean the site itself is trustworthy or safe.

### What HTTPS does *not* hide

An observer on the network can still usually see **which server you connect to** (its IP address and often the site name), and roughly how much data moves. They just cannot read the content.

### Use cases

Pretty much everything on the web now: logins, banking, shopping, email in the browser, APIs, and any page that handles personal data. It is the default.

---

## 5. DNS (Port 53)

**DNS = Domain Name System.** The Internet's phone book: it turns **names** (`www.example.com`) into **IP addresses** (`203.0.113.50`), because computers route by IP, but humans remember names.

| | |
|---|---|
| **Port** | **UDP 53** for normal queries; **TCP 53** for large responses and zone transfers |
| **Encrypted** | **No** by default. Optional: **DoT** (DNS over TLS, port 853) and **DoH** (DNS over HTTPS, port 443) |

### Why UDP?

A DNS lookup is a **tiny question and a tiny answer**. UDP needs no handshake, so it is fast, and if the reply is lost the client just asks again. TCP is used when an answer is too big for one UDP reply (the server sets a "truncated" flag), and for **zone transfers** between DNS servers.

### Common record types

| Record | Maps | Example |
|--------|------|---------|
| **A** | Name to **IPv4** address | `example.com -> 203.0.113.50` |
| **AAAA** | Name to **IPv6** address | `example.com -> 2001:db8::1` |
| **CNAME** | Name to another name (alias) | `www.example.com -> example.com` |
| **MX** | Domain to its **mail** server | `example.com -> mail.example.com` |
| **NS** | Domain to its **name servers** | |
| **TXT** | Free text (email security, domain verification) | |
| **PTR** | **IP to name** (reverse lookup) | |

### How a lookup works

Looking up `www.example.com`:

```
 Your PC (stub resolver)
    |  1. Check browser/OS cache -> hit? done.
    v
 Recursive resolver (your router / ISP / 8.8.8.8)
    |  2. Cache? If not, ask the root servers:        "Who handles .com?"
    |  3. Ask the .com TLD servers:                   "Who handles example.com?"
    |  4. Ask example.com's authoritative server:     "What is www.example.com?"
    |  5. Answer: A 203.0.113.50
    v
 Resolver caches the answer (for its TTL) and replies to your PC
```

- **Caching** makes DNS fast: every answer comes with a **TTL (time to live)** saying how long it may be reused.
- The hierarchy: **root** (`.`), then **TLD** (`.com`, `.org`, `.in`), then the **domain's authoritative servers**.

### Use cases

- Every time you open a website, app, or send an email
- Load balancing and failover (different answers for the same name)
- Email routing (MX records)
- Service discovery inside companies

### Security notes

Plain DNS is readable and can be tampered with (spoofing/cache poisoning). Defenses include **DNSSEC** (signing records), and encrypted DNS (**DoH/DoT**).

---

## 6. DHCP (Ports 67 and 68)

**DHCP = Dynamic Host Configuration Protocol.** It **automatically gives devices their network settings** when they join a network, so nobody has to type IP addresses by hand.

| | |
|---|---|
| **Ports** | **UDP 67** (server) and **UDP 68** (client) |
| **Encrypted** | No |

### What it hands out

| Setting | Example |
|---------|---------|
| **IP address** | `192.168.1.10` |
| **Subnet mask** | `255.255.255.0` |
| **Default gateway** | `192.168.1.1` |
| **DNS server(s)** | `192.168.1.1` |
| **Lease time** | 24 hours |

### The DORA process

A new device has **no IP address yet**, so it must use **broadcast**. That is why DHCP uses UDP: you cannot open a TCP connection to a server you have not found.

```
 Client (no IP yet)                              DHCP server
 MAC AA:AA:AA:AA:AA:01                           (usually your router)
      |                                               |
      |  1. DISCOVER  (broadcast)                     |
      |     0.0.0.0:68 -> 255.255.255.255:67          |
      |     "Is there a DHCP server out there?"       |
      |---------------------------------------------->|
      |                                               |
      |  2. OFFER                                     |
      |     "You can have 192.168.1.10 for 24 hours"  |
      |<----------------------------------------------|
      |                                               |
      |  3. REQUEST  (broadcast)                      |
      |     "I'd like to accept 192.168.1.10"         |
      |---------------------------------------------->|
      |                                               |
      |  4. ACKNOWLEDGE                               |
      |     "Confirmed. Here are mask, gateway, DNS." |
      |<----------------------------------------------|
```

**D**iscover, **O**ffer, **R**equest, **A**cknowledge: **DORA**.

### Leases

The address is **leased**, not owned. The client tries to **renew** when about half the lease time has passed. If a device leaves, its address eventually returns to the pool.

### Other facts

- **DHCP relay:** broadcasts do not cross routers, so a **relay agent** on a router forwards requests to a DHCP server on another network.
- **Reservations:** the server can always give the same IP to a given MAC (handy for printers).
- **If DHCP fails**, Windows and others assign themselves a `169.254.x.x` address (APIPA, see [IPv4 notes](ipv4-basics.md)). That is the sign "I could not reach a DHCP server".
- **IPv6** uses **DHCPv6** (UDP 546/547) and/or SLAAC (see [IPv6 notes](ipv6-basics.md)).

### Use cases

Home Wi-Fi, offices, hotspots, phones joining any network: almost every client device gets its address this way. Servers and routers are usually set **statically**.

### Security note

A **rogue DHCP server** on the network can hand out a malicious gateway or DNS server. Managed switches defend against this with **DHCP snooping**.

---

## 7. FTP (Ports 21 and 20)

**FTP = File Transfer Protocol.** One of the oldest protocols: it uploads and downloads files between a client and a server.

| | |
|---|---|
| **Ports** | **TCP 21** (control) and **TCP 20** (data, in active mode) |
| **Encrypted** | **No**: username, password, and files all travel as plain text |

### Two connections

FTP is unusual because it uses **two separate connections**:

| Connection | Port | Carries |
|------------|------|---------|
| **Control** | 21 | Commands and replies (`USER`, `PASS`, `LIST`, `RETR`...) |
| **Data** | 20 (active) or a random port (passive) | The actual files and directory listings |

| Mode | How the data connection is made | Notes |
|------|---------------------------------|-------|
| **Active** | The **server** connects back to the client (from port 20) | Often blocked by client firewalls and NAT |
| **Passive** | The **client** connects to a port the server announces | The common choice today, and friendlier to firewalls |

### Use cases

- Legacy systems and older web hosting uploads
- Automated file exchange with partners
- Public file archives (anonymous FTP)

### Security and what to use instead

Because FTP is unencrypted, anyone on the path can capture the password. Prefer:

| Alternative | Port | What it is |
|-------------|------|------------|
| **SFTP** | 22 | File transfer **over SSH** (a different protocol, not "FTP with security") |
| **FTPS** | 21, or 990 (implicit) | FTP **wrapped in TLS** |

---

## 8. SSH (Port 22)

**SSH = Secure Shell.** It gives you an **encrypted remote command line** on another computer. It replaced Telnet.

| | |
|---|---|
| **Port** | **TCP 22** |
| **Encrypted** | **Yes** |

### What SSH provides

| Feature | Meaning |
|---------|---------|
| **Encryption** | Commands, output, and passwords cannot be read in transit |
| **Server authentication** | The client verifies the server's **host key**, so you know you reached the right machine |
| **User authentication** | By password, or better, by **public/private key pair** |
| **Integrity** | Tampering is detected |

### Typical use

```
ssh alice@203.0.113.50                 # log in to a remote server
```

The first time you connect, SSH shows the server's **fingerprint** and asks you to confirm; it stores it in `known_hosts` and warns you if it ever changes (which could mean an impostor).

### Key-based login (recommended)

```
ssh-keygen                             # create a key pair (private + public)
ssh-copy-id alice@203.0.113.50         # install the PUBLIC key on the server
ssh alice@203.0.113.50                 # now logs in without sending a password
```

The **private key never leaves your computer**; keep it secret.

### More than a terminal

| Feature | Description |
|---------|-------------|
| **SFTP** | Secure file transfer over the same port 22 |
| **SCP** | Secure file copy |
| **Port forwarding / tunneling** | Carry other traffic through the encrypted SSH connection |
| **Automation** | Used by tools like Git and many deployment scripts |

### Use cases

- Managing Linux servers, cloud machines, and network devices (routers, switches)
- Secure file transfer
- Developer workflows (Git over SSH)
- Safe tunneling to reach internal services

### Security tips

Prefer **key authentication**, disable direct root login and password logins where possible, and keep SSH updated. Because port 22 is constantly scanned by bots, you will see a lot of failed login attempts on any public server. Changing the port only reduces noise; it is not real security.

---

## 9. Telnet (Port 23)

**Telnet** gives you a **remote command line** on another device, just like SSH, but **with no encryption at all**.

| | |
|---|---|
| **Port** | **TCP 23** |
| **Encrypted** | **No**: usernames, passwords, and every keystroke are in clear text |

### Why it still gets taught

- It was the standard remote-login tool for decades, so you will see it in older gear and exam questions.
- It is a **handy way to test whether a TCP port is open**, because it simply opens a TCP connection and lets you type:

```
telnet example.com 80
GET / HTTP/1.0        <- type this, then press Enter twice

HTTP/1.0 200 OK ...   <- the server answers: you just spoke HTTP by hand
```

(Modern Windows has the Telnet *client* turned off by default. You can use `nc example.com 80` as an alternative.)

### Security

Anyone who can capture the traffic sees everything you type, including the password. **Never use Telnet for real administration over an untrusted network. Use SSH instead.** Many networks disable the Telnet service entirely.

### Use cases today

- Legacy devices that support nothing else (often only on isolated management networks)
- **Port testing** (`telnet host port`)
- Learning and lab demonstrations (always on your own lab network)

---

## 10. Insecure vs Secure Pairs

A pattern worth memorizing: most old plaintext protocols have an encrypted successor.

| Insecure (plaintext) | Port | Secure replacement | Port |
|----------------------|------|--------------------|------|
| **HTTP** | 80 | **HTTPS** | 443 |
| **Telnet** | 23 | **SSH** | 22 |
| **FTP** | 21 | **SFTP** (over SSH) / **FTPS** | 22 / 990 |
| **DNS** | 53 | **DNS over TLS / HTTPS** | 853 / 443 |
| **SMTP** | 25 | SMTP with TLS (**587** submission with STARTTLS, **465** implicit TLS) | 587 / 465 |
| **POP3** | 110 | **POP3S** | 995 |
| **IMAP** | 143 | **IMAPS** | 993 |
| **LDAP** | 389 | **LDAPS** | 636 |

**Rule of thumb:** if the protocol sends passwords, it should be encrypted.

---

## 11. More Ports Worth Knowing

| Port | Protocol | Transport | Purpose |
|------|----------|-----------|---------|
| **25** | SMTP | TCP | Mail **between servers** |
| **587** | SMTP submission | TCP | Mail from your **email client** to your server |
| **110** | POP3 | TCP | Download email |
| **143** | IMAP | TCP | Access email on the server |
| **69** | TFTP | UDP | Very simple file transfer (device firmware, network booting) |
| **123** | NTP | UDP | Time synchronization |
| **161 / 162** | SNMP | UDP | Network device monitoring / traps |
| **389** | LDAP | TCP/UDP | Directory services (user lookups) |
| **445** | SMB | TCP | Windows file sharing |
| **514** | Syslog | UDP | Log messages |
| **3389** | RDP | TCP | Windows Remote Desktop |
| **3306** | MySQL | TCP | Database |
| **5432** | PostgreSQL | TCP | Database |
| **8080 / 8443** | HTTP / HTTPS alternates | TCP | Common for dev servers and proxies |

---

## 12. Putting It Together: A Normal Day on the Network

Here is how these protocols cooperate in real life.

| Step | Event | Protocol | Port |
|------|-------|----------|------|
| 1 | Your laptop joins the Wi-Fi and receives IP, mask, gateway, and DNS | **DHCP** | UDP 67/68 |
| 2 | It needs the router's MAC to reach the gateway | ARP (see [ARP notes](arp-and-mac-basics.md)) | Layer 2 |
| 3 | You type `www.example.com`; the browser looks up its IP | **DNS** | UDP 53 |
| 4 | The browser connects (TCP handshake) and encrypts the session | **HTTPS** (TLS) | TCP 443 |
| 5 | The page loads, and an old internal tool on port 80 shows "Not secure" | **HTTP** | TCP 80 |
| 6 | Later, an admin logs in to a server to restart a service | **SSH** | TCP 22 |
| 7 | The same admin uploads a file securely | **SFTP** (over SSH) | TCP 22 |
| 8 | A legacy printer that only supports FTP gets a nightly file | **FTP** | TCP 21 |

Every step above relies on the transport layer from the [TCP vs UDP notes](tcp-udp-transport-basics.md): DNS and DHCP use **UDP** (small, fast, or no IP yet), while web, SSH, FTP, and Telnet use **TCP** (they need reliable, ordered delivery).

---

## 13. Hands-On Lab

Try these on your own computer or lab network.

### Lab 1: DNS

```
nslookup www.example.com            # Windows / macOS / Linux
dig www.example.com                 # Linux / macOS (shows TTL and more detail)
dig www.example.com MX              # mail servers
dig -x 8.8.8.8                      # reverse (PTR) lookup
```

Look at the **TTL** in the answer, then run it again a few seconds later and see if it changes.

### Lab 2: DHCP

```
ipconfig /all                       # Windows: shows DHCP server, lease obtained/expires
ip addr                             # Linux: your IP address
ipconfig /release  &&  ipconfig /renew     # Windows (admin): watch the DORA process in Wireshark
```

In Wireshark, filter on `dhcp` (or `bootp` in older versions) and look for **Discover, Offer, Request, ACK**.

### Lab 3: HTTP vs HTTPS

```
curl -v http://example.com          # port 80: all text, readable
curl -v https://example.com         # port 443: look for the TLS handshake lines
curl -I https://example.com         # headers only (status code)
```

### Lab 4: Test a port by hand

```
telnet example.com 80               # or:  nc example.com 80
GET / HTTP/1.0                      # then press Enter twice
```

### Lab 5: See what is listening on your machine

```
ss -tuln                            # Linux: listening TCP (t) and UDP (u) ports
netstat -an                         # Windows / macOS
```

### Lab 6: SSH

```
ssh user@your-server                # use a machine you own, such as a Raspberry Pi or a cloud VM
```

### Wireshark filters

| Filter | Shows |
|--------|-------|
| `dns` | DNS queries and replies |
| `dhcp` | DHCP exchanges |
| `http` | Plain HTTP |
| `tcp.port == 443` | HTTPS traffic (you will see TLS, not content) |
| `ssh` | SSH traffic (encrypted after the start) |

Only capture on networks you own or have permission to monitor, and only try Telnet or FTP demos in your own lab, never with real passwords.

---

## 14. Practice Questions

1. What are the default ports for HTTP, HTTPS, DNS, DHCP (server and client), FTP, SSH, and Telnet?
2. Which of those protocols use **UDP** by default?
3. Why does DNS normally use UDP? When does it use TCP?
4. Why does DHCP use broadcast and UDP?
5. Spell out the four DHCP steps in order.
6. What is the difference between HTTP and HTTPS?
7. Does the padlock in the browser mean a website is safe to trust?
8. FTP uses two ports. What do they carry?
9. What is the difference between SFTP and FTPS?
10. Give two reasons to use SSH instead of Telnet.
11. Name a legitimate use of Telnet today.
12. A computer shows the IP address `169.254.17.9`. Which protocol most likely failed?
13. Match each to its secure version: HTTP, Telnet, FTP.
14. A DNS answer has a TTL of 300. What does that mean?

<details>
<summary>Show answers</summary>

1. **HTTP 80**, **HTTPS 443**, **DNS 53**, **DHCP 67 (server) / 68 (client)**, **FTP 21 (control) / 20 (data)**, **SSH 22**, **Telnet 23**.
2. **DNS** (normal queries) and **DHCP**. (HTTP/3 also uses UDP 443.) HTTP, HTTPS, FTP, SSH, and Telnet use TCP.
3. A lookup is a tiny request and reply, so UDP is fast and needs no handshake; a lost reply is simply asked again. DNS uses **TCP** for large responses (truncated replies) and zone transfers.
4. The client **has no IP address yet** and does not know the server's address, so it must **broadcast**. You cannot set up a TCP connection to an unknown server, so UDP is used.
5. **Discover, Offer, Request, Acknowledge** (DORA).
6. HTTPS is **HTTP inside TLS**: encrypted, integrity-protected, and the server is authenticated by a certificate. HTTP is plain text.
7. **No.** It means the connection is encrypted and the certificate matches the domain. A malicious site can also have HTTPS.
8. **Port 21** carries **commands/control**; **port 20** (active mode, or a random port in passive mode) carries the **file data**.
9. **SFTP** is a different protocol that runs **over SSH (port 22)**. **FTPS** is the original FTP protocol **wrapped in TLS**.
10. SSH is **encrypted** (Telnet sends passwords in clear text), and SSH **authenticates the server** and supports key-based login, file transfer, and tunneling.
11. **Testing whether a TCP port is open** (`telnet host port`), and managing old devices on isolated lab or management networks.
12. **DHCP.** `169.254.x.x` is a self-assigned (APIPA) address used when no DHCP server answered.
13. HTTP to **HTTPS**, Telnet to **SSH**, FTP to **SFTP or FTPS**.
14. Resolvers may **cache** the answer for **300 seconds (5 minutes)** before asking again.

</details>

---

## 15. Quick Cheat Sheet

| Protocol | Port | Transport | Secure? | Use in one line |
|----------|------|-----------|---------|-----------------|
| **HTTP** | 80 | TCP | No | Web pages and APIs (plain) |
| **HTTPS** | 443 | TCP (+UDP for HTTP/3) | Yes | Secure web pages and APIs |
| **DNS** | 53 | UDP (TCP for big/zone) | No | Name to IP address |
| **DHCP** | 67 / 68 | UDP | No | Auto IP configuration (DORA) |
| **FTP** | 21 / 20 | TCP | No | File transfer (legacy) |
| **SSH** | 22 | TCP | Yes | Secure remote shell, SFTP, tunnels |
| **Telnet** | 23 | TCP | No | Old remote shell; port testing |

| Remember | |
|----------|---|
| Port ranges | 0-1023 well-known, 1024-49151 registered, 49152-65535 ephemeral |
| UDP among these | DNS, DHCP (and HTTP/3 via QUIC) |
| Plaintext vs encrypted | HTTP/Telnet/FTP are plaintext; HTTPS/SSH/SFTP are encrypted |
| DHCP steps | **D**iscover, **O**ffer, **R**equest, **A**cknowledge |
| DNS hierarchy | Resolver, root, TLD, authoritative |
| Number trick | 20/21 FTP, 22 SSH, 23 Telnet |

### Remember in one sentence

> **DHCP gets you an address, DNS turns names into addresses, HTTP/HTTPS fetch web content, SSH and Telnet give remote shells, and FTP moves files, and for each plaintext protocol (HTTP, Telnet, FTP) there is an encrypted replacement (HTTPS, SSH, SFTP) that you should use instead.**

---

*Notes by [your name]. Feel free to open an issue or PR if you spot a mistake.*
