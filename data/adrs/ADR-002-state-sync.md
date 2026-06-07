# ADR-002: State Synchronization Model

Date: 2023-05-02
Status: Accepted
Author: Engine Lead

## Context
Breakline is a fast competitive shooter, so the server must stay authoritative to
resist cheating, while clients must feel responsive despite real-world latency.
We needed a model for how match state moves between the server and clients.

## Options Considered

1. **Authoritative server + snapshots + client prediction**: server simulates, clients predict and reconcile. Standard for shooters and cheat-resistant.
2. **Deterministic lockstep**: send only inputs and simulate identically everywhere. Cheap bandwidth, but one desync stalls the match and it breaks on floating-point drift.
3. **Peer-to-peer with host migration**: no server cost, but the host can cheat and migration is fragile.

## Decision
Authoritative server on a fixed 30 Hz tick. Clients send inputs; the server steps the
custom archetype ECS world and broadcasts a full world snapshot each tick. Clients interpolate
remote entities, predict the local player, and reconcile against the authoritative snapshot.
The server resolves shots with lag-compensated hit registration: it keeps a short per-entity
transform history and rewinds to the shooter's view, so hits land fairly across latency.

## Rationale
- Server authority is the only credible answer to aimbots and movement hacks.
- Prediction and interpolation hide latency without giving up authority.
- Full snapshots were simple to ship for a 5v5 prototype.

## Consequences

**Positive**
- Cheat-resistant by construction; the server is the single source of truth.
- Responsive feel even at 80-120 ms ping.
- Lag-compensated hit registration keeps shots fair across ping without trusting the client.

**Negative**
- A full snapshot to every client each tick is O(entities × clients): bandwidth and
  CPU climb with match size, so 5v5 is the practical ceiling today.
- No area-of-interest filtering and no snapshot delta compression yet; every client
  receives every networked entity.

**To watch**
- Before raising player count or match count, add delta-compressed snapshots,
  area-of-interest filtering, and a hard per-tick CPU budget. Full-state broadcast
  is the scaling wall, not the transport.
