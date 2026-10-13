---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Databases_and_Streams/Event_Sourcing"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Event Sourcing

**L5 Expert** · [↑ Databases and Streams](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In event sourcing the application records immutable events describing what happened as the source of truth, and the current state is merely a view derived by replaying them.

### 1. The big picture

Normally a database stores *current state*: an `UPDATE` overwrites the old value and history is gone unless you add audit tables. Change data capture extracts events *from* a mutable database after the fact, with the log as a low-level detail. **Event sourcing** inverts this: the application's own domain events — "SeatReserved", "ItemAddedToCart", "AddressChanged" — are what is stored, in an append-only log. Current state is whatever you get by applying the events in order.

The idea comes from domain-driven design. It communicates intent (why something changed), supports auditing and debugging, enables new derived views later, and meshes naturally with stream processing: the event log *is* the stream.

### 2. Core concepts and vocabulary

- **Event** — an immutable fact about something that happened, named in the past tense (`OrderPlaced`).
- **Command** — a request to do something (`PlaceOrder`); may be rejected. Becomes an event only after validation.
- **Event store / log** — append-only storage for events, ordered per entity/aggregate.
- **Aggregate** — a unit of consistency (e.g., one order); events are ordered within it.
- **Projection / read model** — state derived from events for querying.
- **Snapshot** — a saved projection at a log position to avoid replaying everything.
- **CQRS** — Command Query Responsibility Segregation: separate write model (events) from read models.
- **Optimistic concurrency** — writes carry the expected version; conflicts are rejected.
- **Upcasting** — converting old event versions to the current schema when reading.

### 3. How it works, step by step

1. A user action arrives as a **command** (`ReserveSeat(seat=12A, user=U1)`).
2. The handler loads the aggregate's current state by **replaying its events** (or snapshot + later events).
3. It validates the command against that state (is seat 12A free?). If invalid, it is rejected — no event written.
4. If valid, it appends an **event** (`SeatReserved{seat=12A,user=U1}`) to the log, with the expected version for concurrency control.
5. Consumers subscribe to the log and update **read models** (seat map, user bookings, analytics, search).
6. If a read model's logic changes or a bug is found, delete it and **rebuild by replaying** the log.

```text
command ──► validate against state ──► append event ──► log (source of truth)
                                                         │
                                  ┌──────────────────────┼───────────────────┐
                                  ▼                      ▼                   ▼
                           seat-map view          user-bookings view     analytics
```

### 4. Worked example

```sql
-- Event table (append-only), one row per event
CREATE TABLE events (
  aggregate_id uuid,
  version      int,
  type         text,
  payload      jsonb,
  occurred_at  timestamptz DEFAULT now(),
  PRIMARY KEY (aggregate_id, version)       -- gives optimistic concurrency
);

-- Command handler (pseudo-flow):
-- 1. SELECT * FROM events WHERE aggregate_id = $1 ORDER BY version;   -- replay
-- 2. decide; new version = last_version + 1
-- 3. INSERT INTO events VALUES ($1, last_version + 1, 'SeatReserved', '{...}');
--    a concurrent writer that used the same version hits the PK -> retry
```

Replay of order #7: `OrderPlaced` → `ItemAdded(sku=A)` → `ItemAdded(sku=B)` → `ItemRemoved(sku=A)` → `OrderPaid` gives state `{items:[B], status: paid}`. A mutable-table design would only show the final state; here you also know the customer removed A before paying — useful for analytics and "why" questions.

### 5. Under the hood

- **Events vs state-mutation logs.** With CDC the log records low-level row changes (the app overwrote a row); with event sourcing the application *writes meaningful events* and builds state from them. Event sourcing events must stay valid forever, so they should state intent and facts, not implementation detail.
- **Commands vs events.** A command may fail validation; an event is a fact that is already true. Validation must happen *before* the event is written; consumers must not reject events. If a consistency rule is needed (e.g., seat uniqueness), check it synchronously when handling the command, or write a tentative event (`SeatRequested`) and later a confirming or cancelling event.
- **Replay cost.** Long-lived aggregates need snapshots. Log compaction is **not** the same: compaction is valid for state-changing CDC events keyed by primary key, but for event sourcing events later events don't override earlier ones, so full history must be kept or snapshotted.
- **Concurrency control.** Version-checked appends (compare-and-set on `(aggregate_id, version)`) avoid lost updates; across aggregates you accept eventual consistency or use sagas.
- **Schema evolution.** Events live forever, so use forward/backward compatible encodings and upcasters; never rewrite history lightly.
- **GDPR/deletion.** Immutable logs conflict with the right to erasure; solutions include crypto-shredding (delete per-user keys), or storing PII outside the log.
- **Read models are disposable.** Because they derive from the log deterministically, a new index or a changed schema is just a new consumer started from offset 0.

### 6. Performance and design implications

- Writes are cheap appends; reads come from precomputed projections, so read scaling is independent.
- Eventual consistency between write and read model: users may not see their own write immediately (read-your-writes may need to wait on the projection).
- Rebuilding large projections takes time; plan for parallel replay.
- Storage grows monotonically; plan retention/snapshots/archival.
- Debugging is easier — you can replay the exact sequence that led to a bug.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `PRIMARY KEY (aggregate_id, version)` | Enforces ordered, conflict-free appends | Relational event store |
| `expectedVersion` (EventStoreDB/Axon) | Optimistic concurrency on append | Preventing lost updates |
| Snapshot every N events | Bounds replay time | Long aggregates |
| Subscription / catch-up consumer | Reads the log to build views | Projections |
| Outbox / log relay | Publishes stored events to a broker | Integrating with other services |
| Upcaster | Migrates old event versions | Schema change |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Audit trail, temporal queries and "why" matter | Simple CRUD with no history needs |
| Many different read models from the same facts | Team is unfamiliar; operational complexity is a risk |
| Rebuilding/derive new views is common | Strong cross-aggregate consistency required synchronously |
| Domain is naturally expressed as events | Strict deletion requirements without a plan (privacy) |

### 9. Common mistakes

- **Events that are just CRUD diffs** (`RowUpdated`) → lose intent → name business facts.
- **Validating in consumers** → events can't be "rejected" → validate at command time.
- **No snapshots** → slow loads → snapshot periodically.
- **Mutating or deleting old events** → breaks replay and trust → append corrections instead.
- **Assuming immediate consistency of read models** → stale reads → design UX for lag or read from the write side.
- **Ignoring schema evolution** → old events unreadable → versioned events and upcasters.

### 10. 🎯 Interview kit

**30-second answer:** Event sourcing stores an append-only log of domain events as the system of record; current state is derived by replaying them, and queries are served from projections. It gives history, auditability and the ability to build new views, at the cost of complexity and eventual consistency.

**Structure a longer answer:** state-based vs event-based storage → commands vs events → write path with version check → projections/snapshots → trade-offs (consistency, deletion, schema evolution).

**Likely follow-ups:**
- *Event sourcing vs CDC?* → App-level intent events vs row-level changes extracted from a mutable DB.
- *How do you handle concurrent updates?* → Expected-version append.
- *How do you delete personal data?* → Crypto-shredding / PII outside the log.
- *Can you use log compaction?* → Not for event-sourced events; snapshots instead.
- *How do you fix a bad projection?* → Drop and rebuild from the log.

**Signals interviewers look for:** distinguishes command vs event, knows replay/snapshot/concurrency/schema issues.

### 11. Check yourself

- [ ] Write the events for a shopping cart and replay them to current state.
- [ ] Explain why a failed validation must not produce an event.
- [ ] Describe how to add a new read model after a year of events.
- [ ] Explain why compaction doesn't fit event-sourced logs.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Event Sourcing › Deriving current state from the event log
- *Designing Data-Intensive Applications* — Martin Kleppmann — Event Sourcing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Event Sourcing › Commands and events
- *Designing Data-Intensive Applications* — Martin Kleppmann — Event Sourcing › Concurrency control
- *Designing Data-Intensive Applications* — Martin Kleppmann — E

