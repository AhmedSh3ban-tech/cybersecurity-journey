# Chapter 5 — TCP/UDP & the Three-Way Handshake
### Study Notes: Professor Messer N10-009 (Week 5 content — Objective 1.4, Transport Layer)

> This is the deep-dive on Layer 4 — the layer you've already touched in every Wireshark capture so far (ports, SYN/ACK flags), but haven't formally unpacked. By the end of this chapter you should be able to explain *why* TCP and UDP exist as two separate protocols, not just that they do.

---

## The core question this chapter answers

**Why does the internet need two different transport protocols instead of one?**

Every application has different requirements. Some need *guaranteed, ordered, error-checked* delivery (a file download — you can't have missing chunks). Some need *speed over guarantees* (a live video call — a dropped frame is fine, waiting for it to be re-sent and cause lag is not). TCP and UDP exist because "reliable" and "fast" are in tension, and no single protocol optimizes for both.

---

## Part 1 — TCP (Transmission Control Protocol)

**Core property: connection-oriented, reliable, ordered.**

TCP guarantees:
1. **Delivery** — if a packet (segment) is lost, it gets retransmitted
2. **Order** — segments are reassembled in the correct order even if they arrive out of order
3. **Error checking** — corrupted data is detected and re-requested
4. **Flow control** — the receiver can tell the sender to slow down if it's overwhelmed

**How it achieves this — sequence numbers and acknowledgments:**

Every byte of data TCP sends is numbered (a sequence number). The receiver acknowledges (ACK) which bytes it has received. If the sender doesn't get an ACK within a timeout window, it assumes the data was lost and retransmits it. This numbering is *also* what lets the receiver reorder segments that arrived out of sequence.

**The cost of all this reliability:** overhead. Every TCP segment carries extra header information (sequence numbers, ACK numbers, flags, window size) and every connection requires setup (the handshake, below) and teardown before/after data transfer. This overhead is the tradeoff for reliability.

---

## Part 2 — The Three-Way Handshake (SYN, SYN-ACK, ACK)

**Why a handshake exists at all:** Before TCP will trust that a connection is real and both sides are ready, it needs both sides to prove they can actually send *and* receive. A one-way "hello" isn't enough — what if the other side never got it, or can't respond?

**The three steps, in mechanism, not just names:**

| Step | Direction | Flag(s) | What's actually happening |
|---|---|---|---|
| 1 | Client → Server | **SYN** | "I want to start a connection. Here's my initial sequence number (let's call it X)." |
| 2 | Server → Client | **SYN, ACK** | "Acknowledged — I got your X (so I'll expect X+1 next). Here's *my* initial sequence number (Y), which you now need to acknowledge." |
| 3 | Client → Server | **ACK** | "Acknowledged — I got your Y (so I'll expect Y+1 next). Connection established." |

**Why this design (why not just 2 steps)?**

A 2-step handshake (SYN → ACK) would only prove the *client* can send and the *server* can receive. It would **not** prove the server can send data back to the client and the client can receive it. The third step exists specifically to confirm the *reverse* direction works too — both sides need to prove they can both send and receive before real data flows. This is why it's called "three-way," and it's the single most commonly misunderstood detail (people memorize "SYN, SYN-ACK, ACK" without knowing *why* three steps, not two).

**What you already saw in your Wireshark lab (Chapter 1 application):** This is exactly the SYN → SYN-ACK → ACK sequence you filtered for with `tcp.flags.syn==1 or tcp.flags.ack==1`. Now you have the full mechanism behind what you observed.

**Connection teardown (the less-discussed but equally real other half):**

TCP also formally closes connections using a **four-way handshake** with FIN (finish) flags:
1. Client → Server: FIN ("I'm done sending")
2. Server → Client: ACK (acknowledges the FIN)
3. Server → Client: FIN ("I'm also done sending")
4. Client → Server: ACK (acknowledges, connection closed)

It's four steps instead of three because each side needs to independently declare "I'm done" — a connection can have one side finish sending while still receiving data from the other.

**Self-check:**
1. Explain, in your own words, why a 2-way handshake wouldn't be sufficient to establish a reliable connection.
2. If you see a packet with only the ACK flag set (no SYN), what does that tell you about where in the handshake process you are?
3. Why does closing a TCP connection need 4 steps while opening one needs only 3?

---

## Part 3 — UDP (User Datagram Protocol)

**Core property: connectionless, unreliable, unordered — "fire and forget."**

UDP does **not**:
- Guarantee delivery (no retransmission if lost)
- Guarantee order (no sequence numbers to reorder with)
- Perform a handshake (no setup — just send)
- Provide flow control

**Why anyone would want this (this is the part people get wrong):** UDP isn't "worse" than TCP — it's optimized for a different priority. For real-time applications (voice calls, live video, online gaming), a dropped or late packet is *useless* even if retransmitted — by the time it arrives, the moment has passed. In these cases, the overhead of TCP's guarantees is pure cost with no benefit; you'd rather skip a lost frame and keep moving than pause and wait for a retransmission that arrives too late to matter.

**UDP is also used where the application itself handles reliability**, or where speed matters more than perfection:
- **DNS** (which you saw in Chapter 4) — a lost DNS query just gets re-sent by the application after a short timeout; the overhead of a full TCP handshake for a tiny, one-shot query would be wasteful
- **DHCP** (also Chapter 4) — same logic; it's a quick broadcast-based exchange, not a sustained connection
- **VoIP, video streaming, online gaming** — speed and low latency outweigh occasional imperfection

**Self-check:**
1. Why would using TCP for a live video call actually make call quality *worse* in some situations, not better?
2. Why does DNS use UDP by default, even though DNS *can* fall back to TCP for certain queries (like large responses)? (This is a genuinely interesting edge case — DNS uses TCP when a response is too large for a single UDP packet, e.g., DNSSEC responses.)

---

## Part 4 — Side-by-Side Comparison (the mental model to keep)

| Property | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (handshake required) | Connectionless (no setup) |
| Reliability | Guaranteed delivery, retransmits lost data | No guarantee, no retransmission |
| Ordering | Guaranteed (sequence numbers) | Not guaranteed |
| Speed | Slower (overhead of guarantees) | Faster (minimal overhead) |
| Header size | Larger (more fields: seq #, ack #, flags, window) | Smaller (just source/dest port, length, checksum) |
| Use cases | Web (HTTP/HTTPS), email (SMTP), file transfer (FTP), SSH | DNS, DHCP, VoIP, video streaming, gaming |

**The one-sentence version you should be able to say from memory:** *"TCP trades speed for reliability through a handshake and constant acknowledgment; UDP trades reliability for speed by skipping all of that overhead — the right choice depends entirely on whether the application can tolerate occasional loss."*

---

## Why this chapter matters beyond Phase 1

- **Wireshark (ongoing):** You'll be reading TCP flags and UDP traffic constantly for the rest of this roadmap. This is foundational literacy, not a one-time topic.
- **Phase 5 (Web Security):** Nearly every web attack you'll study (SQLi, XSS, CSRF) rides on top of TCP (since HTTP/HTTPS is TCP-based). Understanding that the *connection* itself is reliable and ordered — but says nothing about whether the *application data* inside it is safe — is a foundational distinction. TCP being reliable doesn't mean the data is trustworthy; that's an application-layer (Layer 7) problem, which is exactly what web security is about.
- **Phase 7 (Defensive Ops/SOC):** A classic attack pattern, the **SYN flood** (a Denial-of-Service technique), directly abuses the three-way handshake: an attacker sends many SYN packets but never completes the handshake with the final ACK, leaving the server holding many "half-open" connections until it runs out of resources. You cannot understand this attack, or recognize it in a log/SIEM alert later, without today's material being solid.
- **Phase 8 (Offensive Practice):** Port scanning (a first step in almost every offensive engagement) works by sending SYN packets and observing the response — an open port replies SYN-ACK, a closed port replies RST (reset), and no response often means filtered/firewalled. This is literally today's handshake mechanism used as a reconnaissance technique.

**Self-check (connect the dots yourself):**
1. Explain how a SYN flood attack abuses the three-way handshake to exhaust server resources.
2. Explain how a port scanner uses SYN packets and the *lack* of a completed handshake to determine if a port is open, closed, or filtered — without ever actually establishing a real connection.

---

## Practical exercise — extend your Wireshark lab

Using the same setup from your Chapter 1 Wireshark lab:

1. Capture a fresh TCP connection (e.g., `curl http://example.com`).
2. Filter: `tcp.flags.syn==1 or tcp.flags.fin==1 or tcp.flags.ack==1`
3. Identify all 3 handshake packets AND, if the connection closes within your capture window, the 4 teardown packets (FIN/ACK sequence).
4. For at least one packet, click into the TCP header details in the middle pane and find:
   - The **Sequence Number** field
   - The **Acknowledgment Number** field
   - The **Flags** field (expand it — you'll see individual checkboxes for SYN, ACK, FIN, RST, etc.)
5. Now capture a DNS query again (from Chapter 4) and confirm: no handshake packets exist at all, because it's UDP. Compare the header size in the packet details pane — UDP's header section should look noticeably shorter/simpler than TCP's.

**Deliverable:** Add a short section to your `/phase1/wireshark-lab-chapter1.md` file (or create `wireshark-lab-chapter5-tcp-udp.md`) documenting:
- Screenshot or text description of the handshake sequence you captured
- The sequence/ack numbers you observed and how they related to each other (Y = X+1 pattern)
- A one-paragraph explanation, in your own words, of why TCP needs this handshake and UDP doesn't

---

## Self-check summary (don't skip this)

1. From memory, explain all 3 steps of the TCP handshake and *why* each step exists (not just what it's called).
2. From memory, explain why UDP skips the handshake entirely and when that tradeoff makes sense.
3. Explain how a SYN flood works.
4. Explain how a basic port scan uses TCP flags to determine port state.
5. Name two protocols that use UDP and explain *why* each one chose UDP over TCP.

Commit this to `/phase1/chapter5-notes.md`.

---

*Next: Chapter 6 — your final Phase 1 chapter — covers the review week (Week 6) plus the VLAN/switching patch we flagged as a curriculum gap. This closes out Phase 1 before you move into Phase 2 (Linux + Automation).*
