# ADR-003: Server Topology

Date: 2023-03-15
Status: Accepted
Author: Infra Engineer

## Context
We needed to ship the Breakline beta quickly. The team was two gameplay programmers
and one part-time infra engineer. One region was fine at prototype scale.

## Decision
A single authoritative server process on one bare-metal box in one region (EU).
Every live match runs inside that one process. There is no matchmaking service and no router:
clients connect straight to that process, which owns every connected peer.

## Rationale
- Cheapest viable option at beta scale; one box, one deploy.
- No orchestration, no service discovery, no networking between processes.
- The whole game restarts with a single binary relaunch.

## Consequences

**Positive**
- Simple to operate and debug; one process, one log, one box.
- All match state is in local memory, so there is no cross-process coordination.

**Negative**
- Single point of failure: one crash or hardware fault drops every live match at once.
- No horizontal scaling: match state and connections are sticky to this one process,
  so a second process cannot pick up load without coordination.
- Single region means 150-250 ms ping for NA and Asia players.

**To watch**
- Before running more than one server, solve match affinity (which server owns a
  match) and shared session and player state. A matchmaker plus a shared store is
  required; you cannot just add boxes behind a load balancer.
