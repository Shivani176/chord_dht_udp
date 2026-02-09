# Architecture

This implementation follows the classic Chord protocol with practical systems additions for UDP reliability and failure handling.

## High-level Design

```mermaid
flowchart LR
  subgraph Peer["Chord Peer (Node)"]
    A1["Chord Logic\n- join/leave\n- lookup\n- put/get/delete"]
    A2["Routing State\n- successor\n- predecessor\n- finger table"]
    A3["Periodic Maintenance\n- stabilize()\n- fix_fingers()\n- check_predecessor()"]
    A4["Local KV Store\n(in-memory)"]
    A5["UDP Transport\n(send/recv)"]
    A6["Reliability Layer\nACK + timeout + retry"]
  end

  A1 <--> A2
  A1 <--> A4
  A1 <--> A3
  A1 --> A6 --> A5
  A5 --> A6 --> A1

  subgraph Network["UDP Network"]
    N1["Peer 1"]
    N2["Peer 2"]
    N3["Peer 3"]
    N4["Peer 4"]
    N5["Peer 5+"]
  end

  A5 <--> Network
```

## Components

### 1) `node.py` (core)
- Node identity + consistent hashing (SHA-1) in a circular ID space
- RPC-like operations: `join`, `leave`, `put`, `get`, `delete`
- Lookup acceleration using finger tables (logarithmic hops)
- Maintenance routines that periodically repair routing state
- Failure detection and ring repair (predecessor checks + stabilization)

### 2) `chord_cli.py` (interactive CLI)
- Starts a node with `(ip, port)` and optional introducer `(ip, port)` to join an existing ring
- Provides a simple `Chord>` shell to run `put/get/delete` and simulate leaves

### 3) UDP reliability enhancements
Because UDP is connectionless and does not guarantee delivery:
- receivers ACK messages
- senders wait for responses and retry on timeout
- failures trigger repair or user-visible error messages

## Message / Operation Types (conceptual)
- join, notify, stabilize
- put, get, delete
- leave, repair
