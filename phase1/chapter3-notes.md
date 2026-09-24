# Chapter 3 — Subnetting (The Core Skill Block)
### Study Notes: Professor Messer N10-009, Videos 21–24 (Objective 1.7, remainder)

> This is the densest, highest-stakes chapter in Phase 1. Your roadmap's competency gate literally says: "by end of Phase 1, you must subnet by hand under time pressure." That skill is built here. Read this once for understanding, then drill separately (Resource 09) until it's fast — the two are different kinds of practice and neither substitutes for the other.

---

## Video 21 — IPv4 Subnet Masks

**Where we left off (Chapter 2, Video 20):** a subnet mask marks which bits of an IP address are "network" (1s) and which are "host" (0s). This video makes that mechanical: it teaches you to read a subnet mask and immediately know two things — how many networks a range has been split into, and how many hosts each network can hold.

**The core relationship you must internalize:**

> **More network bits (more 1s in the mask) = more networks, but fewer hosts per network.**
> **More host bits (more 0s in the mask) = fewer networks, but more hosts per network.**

This is a *tradeoff*, not two independent numbers. Every bit you take from the host side and give to the network side doubles the number of networks and halves the number of hosts per network. That single sentence is 80% of subnetting intuition.

**Worked example:**

Mask `255.255.255.0` in binary: `11111111.11111111.11111111.00000000`
- 24 network bits, 8 host bits
- Hosts per network = 2^8 − 2 = **254** (why minus 2: one address is reserved as the *network address* itself — all-zero host bits — and one as the *broadcast address* — all-one host bits. Neither can be assigned to a device.)

Now shift one bit from host to network: mask `255.255.255.128`:
- Binary: `...11111111.10000000` → 25 network bits, 7 host bits
- Hosts per network = 2^7 − 2 = **126**
- But now there are **2 networks** instead of 1, carved out of the same original address block.

**CIDR notation** is just a shorthand for the mask: `/24` means 24 network bits (= 255.255.255.0), `/25` means 25 network bits (= 255.255.255.128), etc. You'll see this notation constantly — AWS VPCs, Wireshark filters, everywhere — so get fluent translating `/24` ↔ `255.255.255.0` in your head, not by lookup table.

**Why the "minus 2" rule matters beyond trivia:** This is the single most common subnetting arithmetic mistake — forgetting to subtract network and broadcast addresses. It shows up constantly in interview screening questions ("a /27 subnet has how many usable hosts?" — answer requires remembering minus 2, not just 2^n).

**Self-check:**
1. What is the CIDR notation for `255.255.255.192`? How many host bits does it leave?
2. How many usable hosts does a `/28` network support? Show your subtraction.
3. Why can't the all-zeros or all-ones host address be assigned to a real device?

---

## Video 22 — Classful Subnetting

**The core idea:** Before CIDR (flexible, any-bit-boundary subnetting) existed, IP addressing used rigid **classes** — fixed boundaries for where the network/host split happens, based only on the first few bits of the address.

| Class | First octet range | Default mask | Networks | Hosts per network |
|---|---|---|---|---|
| A | 1–126 | 255.0.0.0 (/8) | Few | Huge (~16.7 million) |
| B | 128–191 | 255.255.0.0 (/16) | Moderate | Large (~65,000) |
| C | 192–223 | 255.255.255.0 (/24) | Many | Small (254) |

(127 is reserved for loopback — `127.0.0.1` is "localhost," which you've almost certainly already used without necessarily connecting it to this class system.)

**Why this matters even though it's "obsolete":** Classful addressing was replaced by **classless** (CIDR) addressing specifically because it wastes address space — a Class C network is *always* exactly 254 hosts whether you need 5 or 250, and a Class B network is *always* ~65,000 whether you need 300 or 60,000. CIDR lets you carve a network to the exact size you need. Understanding classful addressing isn't about using it — it's about understanding **why CIDR was invented and what problem it solved**, which you can't appreciate without seeing the rigid system it replaced.

**The one piece of classful thinking that survived:** Private IP ranges (10.x, 172.16–31.x, 192.168.x) are still described using their original classful category (Class A, B, C private ranges respectively) even though we now subnet them with CIDR freely. You'll see this terminology used loosely in real documentation, so recognize it rather than being confused by it.

**Self-check:**
1. Why is a Class C network's default 254-host size often the *wrong* size for a real subnet, and how does CIDR fix that?
2. `172.20.5.1` — which class would this have belonged to under the old system?

---

## Video 23 — Magic Number Subnetting

**The core idea:** This is the actual fast, practical *technique* for subnetting by hand — the method your roadmap's competency gate is testing when it says "subnet by hand under time pressure." Understand this method deeply; it's the one you'll actually use.

**The "magic number" is simply: 256 minus the interesting octet value of the subnet mask.**

**Worked example — the classic interview-style question:**

*"You have `192.168.1.0/26`. What are the subnet ranges?"*

Step 1: `/26` → binary mask `11111111.11111111.11111111.11000000` → last octet = `192`

Step 2: Magic number = 256 − 192 = **64**

Step 3: This means each subnet block is 64 addresses wide, and blocks start at multiples of 64:
- Subnet 1: `192.168.1.0` – `192.168.1.63` (network: .0, broadcast: .63, usable: .1–.62)
- Subnet 2: `192.168.1.64` – `192.168.1.127`
- Subnet 3: `192.168.1.128` – `192.168.1.191`
- Subnet 4: `192.168.1.192` – `192.168.1.255`

That's it. No binary conversion needed once you have the magic number — just count up by that number to find every block boundary.

**Why this method specifically (vs. pure binary math from Video 19):** Binary math is how you *understand* what's happening. Magic number is how you're *fast* once you understand it — this is the "compiled" version of the same logic, and it's what real network engineers actually use in practice, not full binary expansion every time.

**A slightly harder worked example (host-based, not range-based):**

*"You need a subnet that supports at least 20 hosts. What mask do you need, and how many usable hosts does it actually give you?"*

- 20 hosts needed → need 2^n − 2 ≥ 20 → n=5 gives 2^5−2 = 30 (n=4 gives only 14, too few)
- 5 host bits → 32−5 = 27 network bits → mask is **/27** (255.255.255.224)
- Magic number = 256 − 224 = 32 → subnets are in blocks of 32
- Usable hosts per subnet = **30**

**Self-check (do these by hand, time yourself):**
1. `10.0.0.0/28` — what is the magic number, and list the first 3 subnet ranges.
2. You need a subnet for 100 hosts. What's the minimum-size mask (fewest wasted addresses) that satisfies this, and how many hosts does it actually provide?
3. Given `192.168.5.77/26`, which subnet block does this address belong to, and what's the broadcast address for that block?

---

## Video 24 — Calculating IPv4 Subnets and Hosts

**The core idea:** This video is essentially applied practice — taking everything from Videos 21–23 and answering the two question types you'll be drilled on relentlessly in Week 3:

**Question type 1: "Given a network and a mask, how many usable hosts?"**
→ Formula: 2^(host bits) − 2

**Question type 2: "Given a required number of hosts (or subnets), what mask do I need?"**
→ Find the smallest n such that 2^n − 2 ≥ required hosts, then mask = 32 − n bits

**A realistic scenario worth internalizing (this is the kind of question that shows up in real network design, not just exams):**

*"You have `10.10.0.0/16` and need to create subnets for 4 departments: Engineering (500 hosts), Sales (200 hosts), IT (50 hosts), and Guest WiFi (20 hosts). Design the subnets."*

This is called **VLSM (Variable Length Subnet Masking)** — using *different* mask sizes for different subnets carved from the same block, instead of one uniform size for everyone. This is the realistic, efficient way subnetting is actually done in production networks (as opposed to always using one fixed mask size), and it's the natural conclusion of everything in this chapter: since you now know how to size a mask exactly to a host requirement, you can give each department exactly what it needs instead of wasting address space on a one-size-fits-all mask.

- Engineering needs 500 → n=9 (2^9−2=510) → /23
- Sales needs 200 → n=8 (2^8−2=254) → /24
- IT needs 50 → n=6 (2^6−2=62) → /26
- Guest WiFi needs 20 → n=5 (2^5−2=30) → /27

**Why this matters beyond the exercise:** This is exactly the kind of exercise your roadmap's Packet Tracer lab (Weeks 4–6, building a router→switch→subnets topology) will have you actually implement. It's also the mental model behind AWS VPC subnet design in Phase 6 — when you carve a VPC's CIDR block into public/private subnets, you're doing VLSM, just inside a cloud console instead of on paper.

**Self-check (this is your actual competency gate — time yourself, aim for ~2 min per problem by the end of Week 6):**
1. Design subnets for: Dept A (300 hosts), Dept B (60 hosts), Dept C (10 hosts) from `172.16.0.0/16`. Give the mask and address range for each.
2. Given `192.168.10.130/27`, identify: the subnet's network address, broadcast address, and usable range.
3. You're told a device has IP `10.5.6.9` with mask `255.255.254.0`. What is the network address for this device's subnet?

---

## How to actually pass this chapter's gate

1. **Don't move to Chapter 4 (DNS/DHCP, Week 4) until you can solve magic-number problems without hesitation.** This is the one hard stop in Phase 1 your roadmap is strict about, and rightly so — subnetting fluency is assumed baseline knowledge in every subsequent phase (cloud VPCs, reading network diagrams during incident investigation, Packet Tracer labs).
2. **Use pen and paper for the first 20-30 drill problems** (per Resource 09 in your index) — do not reach for a subnet calculator. The goal is that the magic-number method becomes automatic, not that you can verify an answer.
3. **The realistic failure mode here isn't "I don't understand it conceptually"** — it's "I understand it but I'm slow and make small arithmetic errors under time pressure." That's not a comprehension problem, it's a *reps* problem. Budget for it honestly this week rather than assuming one watch-through is enough.

Commit this to `/phase1/chapter3-notes.md`.

---

*Next: Chapter 4 will cover DNS and DHCP (your Week 4 content) — the two "networking functions" from Chapter 1, Video 3, expanded into full mechanics: how a DNS query actually resolves, and how a device gets an IP address automatically via DHCP's four-step handshake (DORA).*
