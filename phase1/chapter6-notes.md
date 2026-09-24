# Chapter 6 — Review + VLANs & Switching (Phase 1 Closeout)
### Study Notes: Professor Messer N10-009, Domain 2 excerpt (VLAN/Switching patch) + Week 6 review

> This is the final chapter of Phase 1. Two jobs this week: (1) patch the VLAN/switching gap we identified back when you first asked about the playlist — the only Domain 2 content in your entire 38-week roadmap — and (2) consolidate everything from Chapters 1–5 before the Phase 1 competency gate. This is not new sprawling content; treat it as the last piece that completes your Layer 2 picture.

---

## Part 1 — Why VLANs Exist (The Problem First, Then the Solution)

**Recall from Chapter 1, Video 2:** a switch operates at Layer 2, forwarding frames based on MAC address, and by default treats every device plugged into it as part of the same single network (one "broadcast domain").

**The problem this creates at scale:** Imagine an office with 200 devices all plugged into switches, all in one flat network. Every broadcast (ARP requests, DHCP discovers, etc. — recall DORA from Chapter 4) gets sent to *all 200 devices*, even if only 2 of them actually need to talk to each other. Beyond wasted bandwidth, this is also a **security problem**: if every device can technically broadcast-reach every other device, there's no logical separation between, say, the finance department's machines and the guest Wi-Fi.

**The solution — VLAN (Virtual Local Area Network):** A VLAN lets you logically divide a single physical switch (or set of switches) into multiple separate broadcast domains, *without* needing separate physical hardware for each group. Devices in VLAN 10 don't see broadcast traffic from VLAN 20, even if they're plugged into the exact same physical switch.

**The key reframe from Chapter 1:** Back in Video 2, we said a switch operates at Layer 2 using MAC addresses, and *can't* separate networks the way a router (Layer 3) can. VLANs are the exception that makes this more nuanced: **a VLAN is a Layer 2 technique that creates Layer 3-like separation** — it's the switch pretending to be multiple switches. This is genuinely one of the more elegant ideas in networking, worth sitting with rather than just memorizing.

**Self-check:**
1. Without VLANs, why does a single flat network of 200 devices create both a performance problem and a security problem?
2. Why is it accurate to call a VLAN a "Layer 2 technique that creates Layer 3-like separation" rather than just calling it a Layer 3 concept?

---

## Part 2 — How VLANs Actually Work (The Mechanism)

**VLAN tagging — 802.1Q:** When a frame needs to travel between switches while preserving which VLAN it belongs to, the switch inserts a small tag into the Ethernet frame header (a 4-byte field defined by the 802.1Q standard) containing a **VLAN ID** (a number from 1–4094). This tag tells the receiving switch "this frame belongs to VLAN 10" so it knows which broadcast domain to keep it confined to.

**Two port types you must be able to distinguish:**

| Port type | Purpose | Carries |
|---|---|---|
| **Access port** | Connects to an end device (a laptop, printer, etc.) | Untagged traffic for exactly ONE VLAN — the end device has no idea VLANs even exist |
| **Trunk port** | Connects switch-to-switch (or switch-to-router) | Tagged traffic for MULTIPLE VLANs simultaneously — this is how VLAN separation is preserved across multiple switches |

**Why this distinction matters practically:** Your laptop, plugged into an access port, never sees or handles VLAN tags — the switch strips/adds tags transparently. It's only the trunk links *between* switches (or to a router) where tagged frames actually travel. This is a common point of confusion: people think every device needs to "know about" VLANs. They don't — only the switches (and any routing device) need to.

**Self-check:**
1. If your laptop is plugged into an access port assigned to VLAN 10, does your laptop's operating system need any special configuration to work with VLANs? Why or why not?
2. Why would two switches connected to each other almost always use a trunk port rather than an access port for that link?

---

## Part 3 — Inter-VLAN Routing (Connecting the Dots Back to Layer 3)

**The natural question:** If VLANs separate broadcast domains, how does a device in VLAN 10 ever talk to a device in VLAN 20 (e.g., a workstation needing to reach a file server on a different VLAN)?

**Answer: something operating at Layer 3 has to route between them.** Two common approaches:
- **Router-on-a-stick** — a single router interface, configured with sub-interfaces for each VLAN, connected via a trunk port to the switch. The router handles the Layer 3 routing decision between VLANs.
- **Layer 3 switch** — a switch with built-in routing capability, eliminating the need for a separate physical router.

**This is the payoff of understanding both this chapter and Chapter 1 together:** VLANs (Layer 2) create the separation; routing (Layer 3) is what selectively allows traffic between those separated segments when it's actually supposed to happen — the same "router reads Layer 3 to decide where traffic goes" concept from Chapter 1, Video 2, just applied between VLANs instead of between physically separate networks.

**Why this matters for your roadmap specifically:**
- **Phase 6 (AWS/Cloud):** When you build a VPC and split it into subnets, you are doing the cloud equivalent of VLAN segmentation — public subnet, private subnet, each isolated, with routing rules (route tables, security groups) controlling what can cross between them. The mental model is identical even though the implementation is virtualized.
- **Phase 7 (SOC/Defensive):** When investigating lateral movement in an incident (an attacker moving from one compromised machine to others), understanding network segmentation (VLANs) is what lets you reason about *how* an attacker could or couldn't have reached a given system, and why proper segmentation is a real defensive control, not just a performance optimization.
- **Phase 8 (Offensive):** VLAN hopping (a real attack technique where an attacker exploits misconfigured trunk ports to jump between VLANs they shouldn't have access to) is a concept you'll now be equipped to actually understand, not just recognize by name.

**Self-check:**
1. Why can't two devices on different VLANs communicate using only switches, no matter how many switches are involved?
2. Draw the connection (in your own words) between VLAN segmentation and AWS VPC subnet design — what's the same, what's different?

---

## Part 4 — Phase 1 Consolidated Review

Before moving to Phase 2, verify you can do the following **without notes**. This is the actual competency gate — not "I watched the videos," but "I can perform these":

### Conceptual understanding (explain out loud, no notes)
- [ ] Explain all 7 OSI layers and give a real device/protocol example for each
- [ ] Explain encapsulation and de-encapsulation using a concrete packet example
- [ ] Explain the difference between a switch and a router in terms of what address each uses to make decisions
- [ ] Explain DORA (DHCP) and why the Request step is broadcast
- [ ] Explain DNS recursive resolution (resolver → root → TLD → authoritative)
- [ ] Explain the TCP three-way handshake and why it's 3 steps, not 2
- [ ] Explain when you'd choose UDP over TCP and why
- [ ] Explain what a VLAN does and why it's needed at scale
- [ ] Explain the difference between an access port and a trunk port

### Practical ability (do this, don't just describe it)
- [ ] Subnet by hand using the magic number method, under time pressure (~2 min per problem)
- [ ] Given an IP + subnet mask, identify network address, broadcast address, and usable range
- [ ] Capture and correctly label a DNS query in Wireshark (Layer 2/3/4/7)
- [ ] Capture and correctly label a TCP three-way handshake in Wireshark
- [ ] Read `ip a` / `ifconfig` output and identify your assigned IP, subnet, and gateway

### Troubleshooting ability (reason through, don't guess)
- [ ] Given "device can't reach the internet," articulate a logical order of things to check (physical link → DHCP lease → gateway reachability → DNS resolution) and *why* that order makes sense
- [ ] Given a Wireshark capture showing repeated SYN packets with no SYN-ACK response, explain what that suggests

### Ability to apply in a project
- [ ] Your home-lab network diagram (Portfolio Milestone, End of Week 6) should now include: a topology choice (Chapter 2) with justification, subnetting rationale (Chapter 3) showing actual address ranges, and — new from this chapter — a VLAN segmentation plan showing which devices/services would logically be separated and why

---

## Part 5 — Building Your Week 6 Portfolio Deliverable

Your roadmap's Week 6 milestone is: **"Network diagram with subnetting rationale."** Given this chapter's addition, upgrade that deliverable to include VLAN thinking. A strong version looks like:

1. **Topology diagram** (Packet Tracer, draw.io, or even a clean hand-drawn scan) showing: a router, at least 2 switches, and multiple end devices grouped logically (e.g., "Admin," "Guest," "Servers")
2. **VLAN assignment table:**

   | VLAN ID | Name | Purpose | Subnet |
   |---|---|---|---|
   | 10 | Admin | Staff workstations | 192.168.10.0/24 |
   | 20 | Guest | Guest Wi-Fi | 192.168.20.0/24 |
   | 30 | Servers | Internal servers | 192.168.30.0/24 |

3. **Subnetting rationale** — why each VLAN got the address range it did (tie back to Chapter 3's VLSM logic if the sizes differ)
4. **One paragraph** explaining why Guest is separated from Admin/Servers specifically — this is where you demonstrate you understand VLANs as a *security* control, not just an organizational one

Save this as `/phase1/home-lab-network-diagram.md` (plus the actual diagram image), replacing or extending whatever draft you may have started earlier in Phase 1.

---

## Self-check summary (final Phase 1 gate — do not skip)

1. Explain, end to end, what happens when your laptop joins a network and requests a webpage — DHCP lease, DNS resolution, TCP handshake, and (if relevant) VLAN/switching — as one continuous narrative, from memory.
2. If you cannot do #1 fluently, identify *specifically* which chapter's content is the weak link, and revisit that chapter's notes before moving to Phase 2 — don't just push forward and hope it clicks later.

---

## Phase 1 → Phase 2 Transition

With Chapter 6 complete, Phase 1 (Networking, Weeks 2–6) is done. Before starting Phase 2 (Linux + Automation Sprint, Weeks 7–12):

- [ ] Commit all 6 chapters of notes to your GitHub repo under `/phase1/`
- [ ] Commit your Wireshark lab documentation (Chapters 1 and 5 exercises)
- [ ] Commit your home-lab network diagram with VLAN segmentation
- [ ] Take a VM snapshot (per your resource index, Resource 04) — you're about to make your Linux VM your daily driver, so this is a good safety checkpoint before that shift

Phase 2 is a different kind of learning (hands-on CLI drilling via Bandit/TryHackMe rather than lecture-style videos), so this is also a good moment to notice: you're moving from "watch and understand" mode into "struggle and do" mode. That's intentional and matches how your roadmap is structured — don't expect Phase 2 study notes to look like these six chapters; the practical guidance will look more like structured hints for challenges you work through yourself.

---

*Phase 1 complete. Ready for Phase 2 when you are.*
