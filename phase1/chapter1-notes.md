# Chapter 1 — Networking Fundamentals
### Study Notes: Professor Messer N10-009, Videos 1–10 (Objectives 1.1–1.5 intro)

> **Note on ordering:** I'm sequencing these by CompTIA objective number (1.1 → 1.5), which matches your roadmap's Week 2 plan ("OSI Model, TCP/IP model, addressing/subnetting overview"). These notes are written from the standard Network+ curriculum content these titles cover — not a transcript — so you understand the *mechanism*, not just the vocabulary.

---

## Video 1 — Understanding the OSI Model (1.1)

**The core idea:** Networking is a *stack of hand-offs*. No single piece of hardware or software does everything — instead, 7 layers each do one narrow job, and each layer only talks to the layer directly above or below it.

**Analogy:** Think of mailing a letter internationally.
- You write the message (Application layer — the actual content)
- You put it in an envelope, addressed (Network layer — where is it going)
- The postal service hands it between trucks, planes, sorting centers (Physical/Data Link — moving the physical envelope)
- Neither the truck driver nor the sorting machine reads your letter — they only care about the address on the envelope.

That's the whole point of layering: **each layer doesn't need to know what's inside the layer above it.** A switch (Layer 2) doesn't care that your data is an HTTP request — it just reads a MAC address and forwards a frame.

**The 7 layers, top to bottom:**

| # | Layer | Job | Real example |
|---|---|---|---|
| 7 | Application | The actual service/protocol the user or program is using | HTTP, DNS, SMTP |
| 6 | Presentation | Formats/encrypts/compresses data so both ends agree on format | TLS encryption, JPEG/ASCII encoding |
| 5 | Session | Opens, manages, and closes a conversation between two programs | Login session, RPC calls |
| 4 | Transport | End-to-end delivery — reliable or not, and to which *port* | TCP (reliable), UDP (fast, no guarantee) |
| 3 | Network | Logical addressing and routing across networks | IP addresses, routers |
| 2 | Data Link | Physical addressing *within* one local network | MAC addresses, switches |
| 1 | Physical | Literal electrical signals / light / radio waves | Cables, Wi-Fi signals |

Mnemonic (top→bottom): **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing.

**Key mechanism — encapsulation:** When data leaves your device, each layer wraps ("encapsulates") the data from the layer above it in its own header, like nested envelopes. When it arrives, the receiving device unwraps layer by layer (de-encapsulation) in reverse order.

**Why this actually matters to you (not just trivia):** Every tool you'll use in this roadmap organizes information by OSI layer without saying so explicitly:
- Wireshark literally displays a captured packet as stacked layers (Ethernet → IP → TCP → HTTP) — you'll see this in your very first Wireshark lab this same week.
- Attacks are classified by which layer they exploit: ARP spoofing = Layer 2, IP spoofing = Layer 3, SYN flood = Layer 4, SQL injection = Layer 7 (application). When you later learn PortSwigger's web attacks in Phase 5, they're *all* Layer 7 — that's not a coincidence, it's because Layer 7 is where application logic (and its bugs) lives.

**Common misconception to avoid:** People try to memorize the 7 names without understanding *why* there are 7. The honest test of understanding isn't "can you recite the list" — it's "can you point at a real packet and say which layer you're looking at, and explain why splitting the job this way is useful engineering (modularity — you can swap out Wi-Fi for Ethernet at Layer 1/2 without touching anything above Layer 3)."

**Self-check (answer before moving on):**
1. Why can a router route traffic from a Wi-Fi network to an Ethernet network without "understanding" HTTP?
2. If TLS encryption breaks, which layer is malfunctioning?
3. Give one attack per layer 2, 3, 4, 7 (no repeats).

---

## Video 2 — Networking Devices (1.2)

**The core idea:** Different devices operate at different OSI layers, and *that's what determines what information they use to make decisions.*

| Device | Layer | Decides based on | What it can't do |
|---|---|---|---|
| Hub (legacy) | 1 | Nothing — blindly repeats electrical signal to all ports | No addressing awareness at all; everyone sees everyone's traffic |
| Switch | 2 | MAC address (builds a table of which MAC is on which port) | Doesn't understand IP addresses or routing between networks |
| Router | 3 | IP address (uses a routing table to pick the best path) | Doesn't inherently inspect Layer 4+ content unless it's a more advanced device |
| Firewall | 3/4 (or up to 7 for "next-gen") | IP, port, or full application content | Basic firewalls only see IP/port; can't stop attacks hidden inside allowed traffic without deeper (Layer 7) inspection |
| Access Point | 1/2 | Radio signal → frame conversion | Same limits as a switch, wireless version |

**Why this matters:** When you're later reading a network diagram or troubleshooting "why can't these two devices talk," the first question is *which device sits between them and what layer does it operate at*. A switch failure looks different from a router failure. This also directly explains why VLANs (the gap we flagged earlier) exist at Layer 2 — they're a way of telling a switch "treat these ports as if they were on separate physical switches," even though it's Layer 3 (IP subnets) that we usually associate with "separate networks."

**Self-check:**
1. Why can't a simple switch stop two devices in different IP subnets from communicating, even if a firewall rule says they shouldn't?
2. If a hub is basically "dumb," why did the industry replace it entirely with switches?

---

## Video 3 — Networking Functions (1.2)

**The core idea:** Beyond physical devices, there are *functions* — services that any device (physical or virtual) might provide. This is the "software layer" on top of hardware roles.

Common functions covered:
- **DHCP** — automatically assigns IP addresses to devices joining a network (you'll dig deeper into this in Week 4)
- **DNS** — resolves human-readable names to IP addresses (also Week 4)
- **NAT** — translates private IP addresses to a public one so multiple devices can share one internet connection
- **Load balancing** — spreads traffic across multiple servers
- **VPN concentrator** — terminates encrypted tunnels from remote users

**Key distinction to hold onto:** A *device* is physical hardware; a *function* can run on dedicated hardware OR be virtualized/software-based (e.g., a router in a cloud environment is just software running on a hypervisor — this becomes directly relevant in Phase 6 cloud networking).

**Why this matters:** NAT in particular is a concept you'll keep bumping into — it's *why* your home devices all share one public IP, and it's also relevant to security (NAT isn't a firewall, even though it accidentally provides some protection by hiding internal IPs — a common misconception worth correcting now).

**Self-check:**
1. Is NAT a security control? Why or why not?
2. Name one function that could run purely in software with no dedicated physical box.

---

## Video 4 — Cloud Models (1.3)

**The core idea:** "The cloud" isn't one thing — it's a spectrum of *how much infrastructure you manage vs. how much the provider manages.*

| Model | You manage | Provider manages | Example |
|---|---|---|---|
| IaaS (Infrastructure as a Service) | OS, apps, data | Physical servers, networking, virtualization | AWS EC2 |
| PaaS (Platform as a Service) | Just your app + data | OS, runtime, servers | AWS Elastic Beanstalk, Heroku |
| SaaS (Software as a Service) | Just your data/usage | Everything else | Gmail, Salesforce |

There's also a deployment-model axis (separate from the service-model axis above):
- **Public cloud** — shared infrastructure, multiple customers (AWS, Azure, GCP)
- **Private cloud** — dedicated to one organization
- **Hybrid cloud** — mix of both
- **Community cloud** — shared among organizations with common requirements

**Why this matters for you specifically:** Phase 6 of your roadmap is entirely AWS (IaaS-focused: EC2, S3, IAM). Understanding *where the responsibility boundary sits* is the single most important security concept in cloud computing — it's literally called the **Shared Responsibility Model**, and almost every real-world cloud breach (misconfigured S3 buckets, exposed EC2 security groups) happens because someone assumed the provider was responsible for something that was actually the customer's job. This one idea previews your entire Phase 6 misconfiguration project.

**Self-check:**
1. In IaaS, who is responsible for patching the operating system — you or the provider?
2. Why does that answer change under PaaS?

---

## Video 5 — Designing the Cloud (1.3)

**The core idea:** This video extends Cloud Models into *architecture decisions* — things like scalability, elasticity, multi-tenancy, and the tradeoffs of designing systems to run in the cloud rather than on fixed hardware.

Key terms:
- **Elasticity** — automatically scaling resources up/down based on demand (vs. traditional infrastructure where you buy fixed capacity)
- **Multi-tenancy** — multiple customers' workloads run on shared underlying hardware, logically separated
- **Availability zones / regions** — geographic and infrastructure redundancy so a single data center failure doesn't take your service down

**Why this matters:** Multi-tenancy is worth sitting with for a second — it's *why* cloud security is fundamentally an identity and access problem rather than a physical-perimeter problem. You can't put a lock on a server that isn't "yours" in a dedicated physical sense; the isolation is enforced entirely through software (hypervisor isolation, IAM policies). This is the conceptual seed for why Phase 6 spends so much time on IAM rather than, say, physical security.

**Self-check:**
1. If multi-tenancy relies on software isolation instead of physical separation, what does that imply about the *kind* of vulnerability that would be catastrophic in a cloud provider's infrastructure?

---

## Video 6 — Introduction to IP (1.4)

**The core idea:** This is your first real dive into Layer 3. IP (Internet Protocol) is the addressing and routing system that lets devices on *different* networks find each other — as opposed to MAC addresses (Layer 2), which only work within one local network segment.

Two IP versions exist:
- **IPv4** — 32-bit addresses, written as four decimal numbers 0–255 (e.g., 192.168.1.1). ~4.3 billion possible addresses, which is why NAT and private address ranges exist (we ran out).
- **IPv6** — 128-bit addresses, vastly larger space, written in hexadecimal (e.g., 2001:0db8::1)

**Private vs. public addressing:** Certain IPv4 ranges are reserved for private/internal use only and are never routed on the public internet:
- 10.0.0.0 – 10.255.255.255
- 172.16.0.0 – 172.31.255.255
- 192.168.0.0 – 192.168.255.255

Any device using one of these needs NAT (from Video 3) to reach the internet.

**Why this matters:** This is the direct on-ramp to subnetting, which is your Week 3 focus and the single most time-intensive skill in Phase 1. You cannot subnet without first being solid on what an IP address actually represents (a network portion + a host portion — more on that once you hit subnet masks).

**Self-check:**
1. Why can two different companies both use 192.168.1.1 internally without conflict, but not both use the same public IP?
2. What problem does IPv6's larger address space solve that IPv4 fundamentally cannot?

---

## Video 7 — Network Communication (1.4)

**The core idea:** This ties Videos 1 and 6 together — walking through what *actually happens*, layer by layer, when one device sends data to another. This is where encapsulation (from Video 1) gets concrete with real IP/MAC addressing.

**Walkthrough of a single request** (e.g., your laptop loading a webpage):
1. **Application layer**: Your browser generates an HTTP request
2. **Transport layer**: TCP wraps it, assigns a source port (random, e.g., 51000) and destination port (80 or 443)
3. **Network layer**: IP wraps that, adds source IP (your laptop) and destination IP (the web server)
4. **Data Link layer**: Ethernet/Wi-Fi wraps that, adds source MAC (your laptop's NIC) and destination MAC (your default gateway/router — NOT the final server's MAC, since that's usually on a different network)
5. **Physical layer**: Converted to electrical signal/radio waves and sent

**Critical detail people miss:** The destination MAC address changes at every hop (each router along the path swaps it for the next hop's MAC), but the destination IP address stays the same the entire journey. This is *the* key insight that explains why Layer 2 is "local" and Layer 3 is "end-to-end."

**Why this matters:** This is exactly what you'll verify hands-on in your Wireshark lab this week — capture a request, and you'll literally see the destination MAC change to your router's MAC while the destination IP stays as the remote server's IP.

**Self-check:**
1. As a packet crosses 3 routers to reach its destination, how many times does the destination IP change? How many times does the destination MAC change?

---

## Video 8 — Common Ports (1.4)

**The core idea:** A port number identifies *which application/service* on a device should receive the data — since a single IP address might be running many services at once (web server, mail server, SSH, etc. all at the same time).

**Ports to actually know now** (memorize meaning, not just the number):

| Port | Protocol | What it's for |
|---|---|---|
| 20/21 | FTP | File transfer (unencrypted — a security smell if you see it) |
| 22 | SSH | Encrypted remote login/administration |
| 23 | Telnet | Unencrypted remote login (legacy, avoid) |
| 25 | SMTP | Sending email |
| 53 | DNS | Name resolution |
| 67/68 | DHCP | Automatic IP assignment |
| 80 | HTTP | Unencrypted web traffic |
| 443 | HTTPS | Encrypted web traffic |
| 3389 | RDP | Windows remote desktop |

**Port ranges matter:**
- **0–1023**: "well-known ports," reserved for standard services (the ones above)
- **1024–49151**: "registered ports," used by specific applications
- **49152–65535**: "ephemeral ports," randomly assigned to your device as the *source* port for outgoing connections

**Why this matters — directly, right now:** Your Wireshark filters this same week (`tcp.port == 80`, etc.) only make sense once you understand *why* a port number identifies a service. And later, in Phase 5, almost every web vulnerability you'll study assumes you already know the difference between port 80 and 443 without thinking about it.

**Self-check:**
1. If you see traffic to port 23 in a capture, what should that immediately make you suspicious of?
2. Why does the *source* port on your laptop change every time you open a new connection, while the *destination* port (e.g., 443) stays the same?

---

## Video 9 — Other Useful Protocols (1.4)

**The core idea:** Rounding out Layer 4/7 protocols that don't fit neatly into "common ports" but matter for real troubleshooting and security work.

Likely covered (standard N10-009 content for this slot):
- **ICMP** — used by `ping` and `traceroute`; not a port-based protocol, operates at Layer 3, used for diagnostics and error reporting (also frequently abused for reconnaissance/DoS — a favorite exam and interview topic)
- **ARP** — resolves an IP address to a MAC address on a local network (this is the protocol ARP spoofing attacks, mentioned back in Video 1, actually abuse)
- **NTP** — Network Time Protocol, synchronizes clocks across devices (underrated: many security logs and certificate validations break silently if NTP is wrong)

**Why this matters:** ARP is worth sitting with — it's the mechanism that fills in "wait, how does my laptop even know the router's MAC address in Video 7's walkthrough?" Answer: it broadcasts an ARP request ("who has this IP?") and the owning device replies with its MAC. ARP spoofing (a Layer 2 attack) works by lying in that reply.

**Self-check:**
1. Why is ARP a *local-network-only* protocol, unlike DNS which works across the entire internet?
2. If your system clock is wrong (NTP failure), what downstream security mechanism would likely break?

---

## Video 10 — Wireless Networking (1.5, intro)

**The core idea:** First touch on wireless — how Wi-Fi differs from wired Ethernet at Layers 1–2, and the basic vocabulary you'll need before the deeper 1.5 cabling/wireless cluster (your next chapter).

Key concepts to expect:
- **SSID** — the network name you see when you connect
- **Frequency bands** — 2.4GHz (longer range, more interference, slower) vs. 5GHz (shorter range, faster, less interference)
- **Wireless standards** (802.11 a/b/g/n/ac/ax) — each generation trading off speed/range/frequency differently
- **Encryption**: WEP (broken, never use) → WPA → WPA2 → WPA3 (current standard)

**Why this matters:** WEP being "broken" isn't trivia — it's your first concrete example of a *cryptographic* protocol failure you'll actually be able to explain the "why" of once you hit crypto fundamentals later. For now, just anchor the fact: **if you ever see WEP in use anywhere, that's an immediate finding in any security assessment.**

**Self-check:**
1. Why does 5GHz Wi-Fi typically have shorter range than 2.4GHz, given it's "faster"?
2. What's the practical difference in your day-to-day between WPA2 and WPA3 in terms of security guarantees?

---

## How to actually use these notes (not just re-read them)

1. **Don't re-read passively.** Cover the tables, try to redraw the OSI stack and the encapsulation flow from memory.
2. **Answer every self-check question out loud or in writing before checking any source.** If you can't, that's the actual gap — not a video to rewatch, but a concept to sit with.
3. **Do the Wireshark cross-reference** for Videos 1, 7, 8, and 9 this same week — matching a live packet capture to these concepts is what converts "I read this" into "I understand this."
4. Commit this file to your GitHub repo under `/phase1/chapter1-notes.md` — it's your first real portfolio artifact for Phase 1.

---

*Next: Chapter 2 will cover the remaining 1.5 videos (cabling/connectors/Ethernet standards) and 1.6 (topologies/architectures) — the physical-layer cluster that sets up your Week 4–6 Packet Tracer work.*
