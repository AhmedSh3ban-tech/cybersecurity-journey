# Chapter 4 — DNS & DHCP
### Study Notes: Professor Messer N10-009 (Week 4 content — DNS and DHCP mechanics)

> These are the two "networking functions" you first met as names only, back in Chapter 1 (Video 3). This chapter opens them up fully: the actual step-by-step mechanics of how a domain name becomes an IP address, and how a device gets an IP address in the first place without a human typing one in.

---

## Part 1 — DHCP (Dynamic Host Configuration Protocol)

**The problem it solves:** Every device on a network needs an IP address, subnet mask, default gateway, and DNS server address to function. Manually configuring this on every device (static addressing) doesn't scale — DHCP automates it.

**The mechanism — DORA (memorize the sequence, understand *why* it's 4 steps, not 2):**

| Step | Name | Who sends it | What it means |
|---|---|---|---|
| 1 | **D**iscover | Client → broadcast | "Is there a DHCP server on this network? I need an address." |
| 2 | **O**ffer | Server → client | "Here's an available IP address you can use." |
| 3 | **R**equest | Client → broadcast | "I accept that offer" (broadcast, not unicast — see below) |
| 4 | **A**cknowledge | Server → client | "Confirmed, that address is now yours for this lease duration." |

**Why "Request" is broadcast instead of a direct reply to the server (the detail people usually skip past):** If multiple DHCP servers exist on the same network and all sent Offers, the client's broadcast Request lets *every* server see which offer was accepted — so the servers whose offers weren't chosen know to release that IP back into their available pool. If it were unicast, only the chosen server would know, and the other servers would hold that address as "reserved" indefinitely, wasting it.

**Key term — lease:** The assigned IP isn't permanent; it's leased for a set duration (e.g., 24 hours) and must be renewed before expiry, or it gets returned to the pool for someone else. This is why a device that's been off the network for a while sometimes comes back with a *different* IP than before.

**Why this matters practically:** When you set up your home lab (Packet Tracer, Weeks 4–6) and later your Linux VM's networking, DHCP is what's silently handing your machine an address — this demystifies "why does my IP sometimes change" and sets up a real troubleshooting skill: if a device can't get online at all, "check if DHCP is working" is often literally step one, before touching DNS, firewall rules, or cabling.

**Security angle worth knowing now (you'll meet this again in Phase 4/5):** A **rogue DHCP server** — an unauthorized device on the network answering Discover broadcasts with its own (malicious) Offers — can hand out a fake default gateway or DNS server, silently routing a victim's traffic through an attacker-controlled machine. This is a real, low-effort attack technique, and it's *exactly* why DORA's broadcast-based design (built for convenience, not security) has a well-known weakness: nothing in the protocol verifies the offering server is authorized.

**Self-check:**
1. Why does DORA need 4 steps instead of just "Discover → here's your IP"?
2. What real-world symptom would you see on a client machine if a rogue DHCP server won the race against the legitimate one?
3. If a device's DHCP lease expires while it's disconnected from the network, what do you expect to happen when it reconnects?

---

## Part 2 — DNS (Domain Name System)

**The problem it solves:** Humans use names (`google.com`); computers route using IP addresses. DNS is the lookup system that translates one into the other — effectively a distributed, hierarchical phonebook.

**The mechanism — hierarchical resolution, walked step by step:**

Say you type `www.example.com` into a browser for the first time (nothing cached):

1. Your device asks its configured **DNS resolver** (usually your ISP's, or a public one like 8.8.8.8) — "what's the IP for www.example.com?"
2. The resolver doesn't know yet, so it asks a **root server** — "who handles `.com`?"
3. Root server replies with the address of the **.com TLD (Top-Level Domain) server**
4. Resolver asks the TLD server — "who handles `example.com`?"
5. TLD server replies with the address of `example.com`'s **authoritative name server**
6. Resolver asks the authoritative server — "what's the IP for `www.example.com`?"
7. Authoritative server replies with the actual IP address
8. Resolver caches this answer (for a duration set by TTL, see below) and returns it to your device

This is called **recursive resolution** — your device asks one resolver and lets *it* do all the hopping around; the resolver does the "recursive" legwork on your behalf.

**Key term — TTL (Time To Live):** Each DNS record specifies how long it can be cached before it must be looked up again. Short TTL = flexible/fast to change (good if you're migrating servers) but more lookup traffic. Long TTL = efficient caching but slower to propagate changes.

**Common record types (know what each is *for*, not just the letter):**

| Record | Purpose |
|---|---|
| **A** | Maps a name to an IPv4 address |
| **AAAA** | Maps a name to an IPv6 address |
| **CNAME** | An alias — points one name to another name (not directly to an IP) |
| **MX** | Specifies which mail server handles email for a domain |
| **TXT** | Arbitrary text — commonly used for domain verification and, importantly, **SPF/DKIM/DMARC records** (email authentication — a real anti-phishing security mechanism you'll want to remember later) |
| **NS** | Specifies which servers are authoritative for a domain |

**Why this matters — directly, not abstractly:**
- Every single Wireshark capture involving a website starts with a DNS query — you'll see this literally in this week's lab (capturing a DNS resolution end-to-end is explicitly on your Week 4 checklist).
- DNS is one of the most abused protocols in security: **DNS spoofing/cache poisoning** (feeding a resolver a fake answer so it caches malicious data), **DNS tunneling** (smuggling data out of a network disguised as DNS queries, since DNS is almost never blocked by firewalls), and **typosquatting** (registering `gooogle.com` to catch mistyped traffic) all exploit trust in this system.
- TXT records for SPF/DKIM/DMARC are your first real touchpoint with *email security*, which becomes relevant again if you ever look at phishing/BEC (business email compromise) in a SOC context (Phase 7).

**Self-check:**
1. Walk through, from memory, what happens between typing a URL and your browser getting an IP address back — don't skip the root/TLD/authoritative steps.
2. Why would an attacker prefer DNS tunneling over just sending data directly over HTTP, from a firewall-evasion standpoint?
3. What's the practical difference between an A record and a CNAME record — when would you use one over the other? (Hint: think about what happens if the underlying IP changes.)
4. Why does a very short TTL create more DNS traffic overall?

---

## Connecting DHCP and DNS together

Here's the piece that ties this whole chapter back to Chapter 1's OSI walkthrough (Video 7): when your laptop joins a network, **DHCP is what tells it which DNS server to use** — the DHCP Offer/Ack doesn't just hand over an IP address; it typically also hands over the DNS server address as one of its options. So the very first thing your machine does after getting an IP (via DORA) is immediately capable of resolving names (via the DNS server DHCP just gave it). These two protocols are functionally chained in the first few seconds a device is on a network — which is exactly what you'll observe if you capture your own laptop connecting to Wi-Fi in Wireshark.

---

## Practical exercise for this chapter (do this, don't just read)

1. In your Linux VM, run `ip a` (or `ifconfig`) and identify your currently leased IP, then check `/etc/resolv.conf` or `resolvectl status` to see which DNS server your DHCP lease handed you.
2. In Wireshark, filter `dns` and refresh a website you haven't visited recently (or flush your DNS cache first: `sudo systemd-resolve --flush-caches` or equivalent). Capture the actual query and response — identify the record type (A) and the TTL value in the response.
3. Run `dig example.com` (or `nslookup` if `dig` isn't installed) and manually trace: which server answered, and is it authoritative or just your resolver's cached answer? (`dig +trace example.com` will literally show you the root → TLD → authoritative walk.)

---

## Self-check summary (answer all before moving on)

1. Explain DORA's four steps and *why* Request is broadcast, from memory, out loud.
2. Explain DNS's recursive resolution chain (resolver → root → TLD → authoritative) from memory.
3. What security risk is created by DHCP having no built-in server authentication?
4. Name two ways DNS gets abused by attackers, and briefly explain the mechanism of each.
5. Where do these two protocols intersect in a device's first few seconds on a network?

Commit this to `/phase1/chapter4-notes.md`.

---

*Next: Chapter 5 covers TCP/UDP, the three-way handshake, and rounds out the port/protocol knowledge from Chapter 1 — this is your Week 5 content, and it directly sets up the Wireshark TCP handshake capture on your Week 5 checklist.*
