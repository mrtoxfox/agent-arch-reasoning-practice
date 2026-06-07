# Tidebreak Studios: Company Context

## What We Do
Tidebreak Studios makes Breakline, a session-based competitive online shooter:
two teams of five drop into a match and the last team standing wins the round.
Fast rounds, tight aim, ranked ladders.
Founded in 2023, single-region launch, growing through closed beta.

## Current Numbers
- 26,000 registered players (closed beta)
- 1,800 peak concurrent players
- ~180 live matches at peak (5v5)
- Server tick rate: 30 Hz
- Average match length: 9 min
- 64% of matches are ranked (competitive integrity matters)

## Growth Trajectory
- Wishlists growing 20% month over month since the beta trailer
- Open beta and paid launch planned for Q4
- Best case at launch: north of 20,000 concurrent players if the tech holds

## Current Technology Stack
- Server: single authoritative C++ game server (fixed 30 Hz tick, custom archetype ECS world)
- Transport: in-house UDP reliability layer (custom acks, retransmit, channels)
- State sync: full world snapshots, client prediction and reconciliation, lag-compensated hit registration
- Client: C++ game client
- Hosting: one bare-metal box, one region (EU)

## Engineering Team
- 2 gameplay / engine programmers (C++)
- 1 client programmer
- 1 part-time backend / infra engineer
- No dedicated netcode-at-scale or SRE experience

## Infrastructure Budget
- Current spend: ~€300/month
- Approved budget for scaling: up to €1,500/month additional

## Known Pain Points (Business Impact)

1. **One process holds every match.** A single server runs all live matches.
   At peak it is CPU-bound, tick time spikes, and players rubber-band.

2. **One crash drops everyone.** A single fault ends every live match at once.
   No isolation between matches, no failover.

3. **No matchmaking.** Players join through a basic queue with no skill matching
   and no routing to a free server. Ranked feels unfair.

4. **Single region.** One EU box. Players in NA and Asia see 150-250 ms ping.
   This is the number one community complaint.

5. **Bandwidth grows with players.** The server sends a full snapshot to every
   client each tick. Bandwidth and CPU climb with match size, so 5v5 is the ceiling.

6. **Hand-rolled transport.** The in-house UDP layer has no congestion control
   and no payload encryption (only a salt-based connection check, no packet crypto).
   Packet loss causes hitches, and the team owns every bug.

## Timeline
- Open beta and paid launch: Q4, hard deadline, marketing committed.
- Fleet plus matchmaking MVP: required before launch, team lead is pushing hard.
- Cross-region play: backlog, heavy community pressure, no firm date.
