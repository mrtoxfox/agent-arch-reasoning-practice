# ADR-001: Networking Transport

Date: 2023-03-15
Status: Accepted
Author: Engine Lead

## Context
The Breakline prototype needed to sync player movement and shots in real time.
The team knew Berkeley sockets and wanted full control over the wire format.
No netcode-at-scale experience, and a hard prototype deadline.

## Decision
Hand-roll a UDP reliability layer in C++: our own packet headers, sequence numbers,
acks, RTT-based retransmit, and reliable-ordered / unreliable-ordered / unreliable-unordered channels.
No third-party transport (ENet, yojimbo, GameNetworkingSockets) was evaluated.

## Rationale
- Full control of every byte on the wire; easy to add fields as the game changed.
- No external dependency to vendor, build, or learn.
- Fast to stand up for a prototype with two players on a LAN.

## Consequences

**Positive**
- The team understands the whole transport; debugging is a single codebase.
- Small, dependency-free, easy to drop into both the client and the server.

**Negative**
- No congestion control: under packet loss the layer keeps retransmitting and makes hitches worse.
- No payload encryption: packets are plaintext and sniffable. Connections are gated by a salt-XOR prefix (clientSalt ^ serverSalt), but there is no packet crypto or integrity MAC, so contents can be read and tampered with.
- Reordering, duplicate acks, and MTU edge cases are under-tested and ours to fix.

**To watch**
- Re-evaluate against ENet or GameNetworkingSockets before scaling past the beta.
- Reliability bugs surface under real-world loss, not on the LAN we tested on.
