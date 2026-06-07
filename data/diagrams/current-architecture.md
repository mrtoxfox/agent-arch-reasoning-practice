# Current Architecture (Breakline game server)

Detailed view of the current server, built on the real Network-Library shape. The three highlighted zones are the ADR traps the workshop's scaling work would change; everything else stays as is.

- 🟠 **orange** = the hand-rolled UDP transport: no congestion control, no payload encryption (ADR-001; Sam's fairness worry, Dana's maintenance burden)
- 🔵 **blue** = full-snapshot replication, the bandwidth wall: no delta, no interest management (ADR-002; Sam's game feel under load)
- 🔴 **red** = single process, one region, no matchmaker: the single point of failure and the reason you cannot just add servers (ADR-003; Dana's ops, Marco's launch and ping complaints)

```mermaid
graph TD
    subgraph Client["Game Client (C++)"]
        direction TB
        C1["Sample input, send to server"]
        C2["Predict local player, reconcile to snapshot"]
        C3["Interpolate remote entities"]
        C1 ~~~ C2 ~~~ C3
    end

    subgraph Net["In-house UDP transport, hand-rolled (client and server share it)"]
        direction TB
        N1["Raw UDP socket (Berkeley sockets)"]
        N2["Connection pipeline<br>salt handshake (clientSalt XOR serverSalt)"]
        N3["Channels<br>reliable-ordered / unreliable-ordered / unreliable-unordered"]
        N4["Acks, sequence numbers, RTT-based retransmit<br>no congestion control, no payload encryption"]
        N1 ~~~ N2 ~~~ N3 ~~~ N4
    end

    subgraph Box["Single authoritative server process: one box, one region (EU)"]
        direction TB
        S1["Fixed 30 Hz tick loop<br>PreTick → Tick → PosTick"]
        S2["Custom archetype ECS world<br>all live match state in memory"]
        S3["Server systems<br>player simulation, lag-compensated hit reg (transform history)"]
        S4["Replication manager<br>full world snapshot to every client, every tick"]
        S1 --> S2 --> S3 --> S4
    end

    Missing["Missing for scale: matchmaker, router, shared session store, second region.<br>A second server cannot pick up load: state and connections are sticky to this one process."]

    Client -->|inputs| Net
    Net -->|inputs| Box
    Box -->|full snapshots| Net
    Net -->|snapshots| Client
    Box -.-> Missing

    subgraph Legend["Workshop change zones (the three ADR traps)"]
        direction TB
        L1["Transport: hand-rolled UDP, no congestion control or encryption (ADR-001)"]
        L2["State sync: full-snapshot bandwidth wall, no delta or interest management (ADR-002)"]
        L3["Topology: single process, one region, no matchmaker (ADR-003)"]
        L1 ~~~ L2 ~~~ L3
    end

    classDef transport fill:#fdeccd,stroke:#e65100,stroke-width:3px;
    classDef statesync fill:#e3f0fb,stroke:#1565c0,stroke-width:3px;
    classDef topology fill:#ffe0e0,stroke:#cc0000,stroke-width:3px;

    class N4,L1 transport;
    class S4,L2 statesync;
    class Missing,L3 topology;

    style Net fill:#fff6ec,stroke:#e65100,stroke-width:2px,stroke-dasharray:5 3;
    style Box fill:#fff0f0,stroke:#cc0000,stroke-width:2px;
```
