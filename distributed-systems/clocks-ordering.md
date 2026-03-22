# Clocks & Ordering

In a distributed system there's no global clock. Each node has its own clock, and they drift. This makes it impossible to determine the exact order of events across nodes using wall-clock time alone. Distributed clocks solve this — they provide a way to reason about event ordering without synchronized physical clocks.

---

## The Ordering Problem

Consider two events:
- Node A writes `x = 1` at timestamp 10:00:00.001
- Node B writes `x = 2` at timestamp 10:00:00.000

Which happened first? You can't tell — Node A's clock might be 5ms ahead. Wall-clock timestamps are unreliable for ordering unless clocks are tightly synchronized (which is expensive and imperfect).

---

## Happened-Before Relation

Leslie Lamport (1978) defined a **partial order** on events without relying on physical clocks:

- **Rule 1:** If events `a` and `b` occur on the same node, and `a` comes before `b`, then `a → b`.
- **Rule 2:** If `a` is a message send and `b` is the corresponding receive, then `a → b`.
- **Rule 3:** If `a → b` and `b → c`, then `a → c` (transitivity).

If neither `a → b` nor `b → a`, the events are **concurrent** — there's no causal relationship and no way to determine which happened first.

---

## Lamport Clocks

A single counter per node that provides a **total order** consistent with the happened-before relation.

**Rules:**
1. Before each event, increment the counter: `clock += 1`
2. When sending a message, include the counter value.
3. When receiving a message, set `clock = max(local_clock, received_clock) + 1`

```
Node A: [1] ──send(1)──→ [2]          [5]
Node B:      [1] [2] ──send(2)──→ [3] ──receive──→ [6]
Node C:                [1] ──receive(2)──→ [3] [4]
```

**Limitation:** Lamport clocks give a total order, but you can't tell from the clock values alone whether two events are causally related or concurrent. If `L(a) < L(b)`, it doesn't mean `a → b` — it only means `a` didn't happen after `b`.

---

## Vector Clocks

Vector clocks fix Lamport's limitation by tracking causality precisely. Each node maintains a vector of counters — one per node in the system.

**Rules:**
1. Before each event, node `i` increments `VC[i] += 1`.
2. When sending, include the full vector.
3. When receiving, merge: `VC[j] = max(VC[j], received_VC[j])` for all j, then increment own entry.

**Comparing vectors:**
- `VC(a) < VC(b)` (a happened before b) — if every entry in `a` ≤ corresponding entry in `b`, and at least one is strictly less.
- **Concurrent** — if neither `VC(a) ≤ VC(b)` nor `VC(b) ≤ VC(a)`.

```
Node A: [1,0,0] → [2,0,0] ──send──→
Node B:            [0,1,0] ──receive──→ [2,2,0] → [2,3,0]
Node C:                      [0,0,1]                         (concurrent with B's events)
```

**Trade-off:** Vector clocks grow with the number of nodes. For large systems, this can become impractical — **dotted version vectors** and **interval tree clocks** are optimizations that bound the size.

**Used by:** Amazon Dynamo (original), Riak

---

## Hybrid Logical Clocks (HLC)

HLCs combine physical timestamps with logical counters to get the best of both worlds:
- **Causality tracking** like Lamport clocks.
- **Close to wall-clock time** for human-readable timestamps.
- **Bounded size** — just a (physical_time, logical_counter) pair, not a vector.

**How it works:**
1. On local event or send: `pt = max(local_physical_time, hlc.pt)`, then increment logical counter if pt didn't change, or reset to 0 if it did.
2. On receive: `pt = max(local_physical_time, hlc.pt, msg.pt)`, adjust counter accordingly.

**Used by:** CockroachDB, MongoDB, YugabyteDB

---

## Google TrueTime

Google's Spanner uses **GPS and atomic clocks** in every datacenter to provide a globally synchronized clock with a known uncertainty bound.

TrueTime returns an interval `[earliest, latest]` instead of a single timestamp. The API:
- `TT.now()` → returns `[earliest, latest]` — the true time is guaranteed to be within this interval.
- `TT.after(t)` → true if `t` is definitely in the past.
- `TT.before(t)` → true if `t` is definitely in the future.

Spanner uses **commit-wait**: after assigning a timestamp to a transaction, it waits out the uncertainty interval before making the result visible. This guarantees **external consistency** (linearizability) using real-time ordering.

**Trade-off:** Requires specialized hardware (GPS + atomic clocks). Not available outside Google's infrastructure (though CockroachDB approximates it with NTP + HLC).

---

## Comparison

| Clock Type | Tracks Causality | Detects Concurrency | Size | Needs Sync | Used By |
|-----------|-----------------|--------------------|----|-----------|---------|
| Lamport | Partial (one direction) | No | O(1) | No | General theory |
| Vector | Full | Yes | O(n) | No | Dynamo, Riak |
| HLC | Full | Partial | O(1) | Loose (NTP) | CockroachDB, MongoDB |
| TrueTime | Via real time | Via real time | O(1) | Tight (GPS+atomic) | Google Spanner |

---

## Interview Tips

- Be able to explain **why** wall-clock time is unreliable (clock drift, NTP jumps, leap seconds).
- Know the happened-before relation and draw a Lamport clock example.
- Understand when you need vector clocks (conflict detection in leaderless replication) vs when Lamport clocks suffice (total ordering for logs).
- Spanner/TrueTime is a great example of trading hardware cost for strong consistency guarantees.

---

## Practice Questions

- Two writes arrive at a leaderless database with conflicting values. How would vector clocks help determine if one causally depends on the other?
- Why can't Lamport clocks detect concurrent events?
- Explain the commit-wait mechanism in Google Spanner. What happens if the uncertainty interval is large?
- Design an ordering system for a distributed event log. Which clock type would you choose and why?
