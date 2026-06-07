# Reference Code: Network-Library (reading map)

The reference codebase is **DanielJimenezMorales/Network-Library**, vendored as a git submodule
at `reference-code/network-library` (MIT, pinned commit). It is a real C++ authoritative game
server: a custom archetype ECS world, a fixed-tick loop, a hand-rolled UDP reliability layer, and snapshot
replication with client prediction and reconciliation. Tidebreak's Breakline server is built on
exactly this shape, so the ADR traps you read in `data/adrs/` are visible in this code.

You do not need to build it (the build is premake5 / Visual Studio 2022). Read the files below.

## Where each ADR trap lives in the code

### ADR-001 (Networking Transport): hand-rolled UDP, no proven library
- `network-library/NetworkLibrary/src/Core/Socket.h` and `Socket.cpp`: the raw UDP socket.
- `network-library/NetworkLibrary/src/transmission_channels/`: the in-house reliability layer.
  See `reliable_ordered_channel.h`, `unreliable_ordered_transmission_channel.h`, and
  `transmission_channel.h`. Sequence numbers, acks, and retransmit are all written by hand.
- What is missing is the point: no congestion control, no payload encryption. This is ADR-001's gap.

### ADR-002 (State Synchronization): authoritative tick + full replication, no interest management
- `network-library/Engine/src/Game.cpp`, lines ~35-53: the fixed-timestep loop.
  `while (accumulator >= FIXED_FRAME_TARGET_DURATION) { PreTick(); Tick(); PosTick(); }`.
  This is the authoritative server tick.
- `network-library/Engine/src/ecs/`: the ECS world (`entity_container`, `archetype`,
  `system_coordinator`). All match state lives here, in memory.
- `network-library/DemoGame/src/server/`: the server-authoritative game systems
  (`player_simulation/`, `hit_reg/` with `server_transform_history_component` for lag-compensated
  rewind, `systems/`). The server simulates; clients send inputs, predict, and reconcile.
- `network-library/NetworkLibrary/src/replication/`: snapshot replication
  (`replication_manager`, `network_entity_storage`, `network_variable*`). Every connected client
  receives every networked entity. There is no area-of-interest filtering, so bandwidth and CPU
  grow with entities times clients. This is ADR-002's hinge: the scaling wall.

### ADR-003 (Server Topology): one process, one region, sticky sessions
- `network-library/NetworkLibrary/src/Core/Server.cpp` and `remote_peers_handler.*`: one server
  process owns every connected peer and all match state in local memory.
- There is no matchmaker, no router, and no shared store. A second process cannot pick up a match,
  because the match state and connections are sticky to this one process. This is ADR-003's trap:
  you cannot scale by just adding boxes.

## Honest gaps in the reference (these are the lesson, not bugs)
- Delta-compressed snapshots and RPCs are on the project roadmap but not implemented. That absence
  is exactly the bandwidth problem ADR-002 flags.
- The library targets a single authoritative process. Fleet, matchmaking, and cross-region routing
  do not exist here. That absence is exactly what ADR-003 says you must design before scaling.

## Updating the submodule
```
git submodule update --init --recursive
```
The submodule is pinned to a specific commit so the reading map stays valid.
