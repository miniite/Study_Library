---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/03_Transactions_and_Consistency/ACID_and_Isolation/Write_Skew_and_Phantoms"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Write Skew and Phantoms

**L5 Expert** · [↑ ACID and Weak Isolation Levels](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Write skew happens when two concurrent transactions each check a condition, then write *different* rows that together break it; snapshot isolation doesn't stop it, and the root cause is usually a phantom — a write that changes the result of someone else's search.

### 1. The big picture
Dirty writes and lost updates are conflicts on the *same* object, and databases have tools for them (row locks, atomic operations, automatic detection under snapshot isolation). Write skew is subtler: the transactions touch **different** rows, nothing looks like a conflict, and yet an invariant that spans rows is violated. It appears in any "check, then act" logic and survives snapshot isolation, PostgreSQL repeatable read, InnoDB repeatable read and Oracle "serializable". Only true serializable isolation, or manual safeguards, stop it.

This lesson builds on snapshot isolation and motivates the serializability techniques in the sibling topic.

### 2. Core concepts and vocabulary
- **Write skew** — two transactions read the same data, then update *different* objects based on what they read, producing a state neither would have allowed alone.
- **Phantom** — a write in one transaction changes the set of rows matching another transaction's search condition.
- **Materializing conflicts** — creating artificial lock rows so a phantom becomes a lock conflict on a real row.
- **`SELECT … FOR UPDATE`** — locks every row the query returns until commit.
- **Precondition / premise** — the fact (e.g. "≥2 doctors on call") that a decision relied on.
- **Multi-object constraint** — an invariant spanning several rows (hard to declare in most databases).

### 3. How it works, step by step
The classic story: a hospital needs at least one doctor on call. Alice and Bob are the two on-call doctors, both feel ill and both click "go off call" at the same time.

1. Alice's transaction: `SELECT count(*) FROM doctors WHERE on_call = true AND shift_id = 1234` → 2.
2. Bob's transaction (concurrently, same snapshot state): same query → 2.
3. Each reasons "two on call, so I can leave" and updates **their own row** to `on_call = false`.
4. No row was written by both, so no write-write conflict is detected. Both commit.
5. Result: zero doctors on call.

Run serially, the second transaction would have seen count = 1 and been refused. Only concurrency made it possible — that is what makes it a race condition.

The **general pattern** behind every write-skew example:
1. A `SELECT` checks a requirement over rows matching a condition.
2. Application code decides to proceed based on the result.
3. A write (`INSERT`/`UPDATE`/`DELETE`) goes ahead and *changes the answer* to the step-1 query.

### 4. Worked example
Meeting-room double booking, unsafe under snapshot isolation:
```sql
BEGIN;
SELECT count(*) FROM bookings
 WHERE room_id = 123
   AND end_time   > '2026-01-01 12:00'
   AND start_time < '2026-01-01 13:00';        -- returns 0 in both sessions
INSERT INTO bookings(room_id,start_time,end_time,user_id)
 VALUES (123,'2026-01-01 12:00','2026-01-01 13:00',666);
COMMIT;                                         -- both sessions succeed -> double booking
```
`FOR UPDATE` cannot help: the query returned *no rows*, so there is nothing to lock. The conflicting object is a **phantom** — a row that doesn't exist yet.

Fixes, in order of preference:
```sql
-- 1) Real serializability
BEGIN ISOLATION LEVEL SERIALIZABLE;   -- PostgreSQL SSI aborts one session; retry it

-- 2) A constraint the database can enforce (PostgreSQL exclusion constraint)
ALTER TABLE bookings ADD CONSTRAINT no_overlap
  EXCLUDE USING gist (room_id WITH =, tstzrange(start_time,end_time) WITH &&);

-- 3) Materialized conflicts: pre-created room×slot rows to lock
BEGIN;
SELECT 1 FROM room_slots
 WHERE room_id = 123 AND slot BETWEEN '12:00' AND '12:45' FOR UPDATE;
-- now check bookings and insert
COMMIT;
```
For the doctor example, where the rows being changed *were* returned by the check, `SELECT … FOR UPDATE` on the on-call rows works.

### 5. Under the hood
- **Why detection of lost updates doesn't cover it:** the automatic check compares two writes to the *same* row version. Write skew has disjoint write sets; the dependency is read-to-write ("my read was invalidated by your write"), which SI doesn't track. Serializable snapshot isolation adds exactly that: it records when a transaction's snapshot read was made stale by a later commit and aborts at commit time.
- **Phantoms vs locks:** locking only protects existing rows. Pessimistic serializable implementations lock *index ranges* or predicates to cover rows that might appear; SSI instead tracks reads and checks at commit.
- **Which cases have simpler fixes?**
  - Username uniqueness → a `UNIQUE` constraint; the second inserter fails.
  - Double spending / balance never negative → a single atomic `UPDATE … SET bal = bal - x WHERE bal >= x`, or a `CHECK` on a row-level aggregate maintained by the DB.
  - Cross-row invariants without built-in support → triggers or materialized views (engine dependent).
- **Materializing conflicts** leaks concurrency control into the data model and is easy to get wrong (forget one code path and the lock is bypassed).

### 6. Performance and design implications
- Serializable costs aborts under contention; budget for retries and keep transactions short.
- Row locks from `FOR UPDATE` cost blocking and deadlock risk; always lock in a consistent order.
- Constraints (unique, exclusion) are the cheapest, most reliable fix because the engine checks them atomically.
- Test with concurrent load — these bugs are timing-dependent and rarely show in unit tests.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SELECT … FOR UPDATE` | Lock returned rows | Check-then-update on existing rows |
| `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE` | Full protection | Any multi-row invariant you can't constrain |
| `UNIQUE` / `EXCLUDE` constraints | DB-enforced rules | Names, non-overlapping ranges |
| `UPDATE … WHERE cond` with row count check | Atomic conditional write | Balances, stock |
| Retry on SQLSTATE `40001` / `40P01` | Handle serialization failure / deadlock | Always with serializable or locks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Declarative constraint | The invariant spans arbitrary rows/tables |
| Serializable isolation | You can't retry or contention is extreme |
| `FOR UPDATE` on existing rows | The check is for the *absence* of rows (phantoms) |
| Materialized conflicts | Any other option exists — it is a last resort |

### 9. Common mistakes
- Assuming `REPEATABLE READ` or SI prevents it → double bookings, negative balances → serializable or constraints.
- Using `FOR UPDATE` on a query that returns zero rows → no lock held → materialize or serialize.
- Checking in application code only → races across app servers → push the rule into the database.
- Forgetting to retry aborted serializable transactions → spurious user errors.

### 10. 🎯 Interview kit
**30-second answer:** Write skew is when two transactions read overlapping data, then write different rows based on it, violating a multi-row invariant — e.g. two on-call doctors both leaving. Snapshot isolation doesn't catch it because there's no same-row conflict; you need serializable isolation, locks, or constraints.

**Structure a longer answer:** clarify (lost update vs write skew) → the select/decide/write pattern → phantom explanation → options ranked: constraint, serializable, FOR UPDATE, materialized conflicts → scenario.

**Likely follow-ups:**
- *Why doesn't FOR UPDATE fix booking systems?* → Nothing to lock when the query returns no rows.
- *Is write skew a generalisation of lost update?* → Yes; same-object case gives dirty write or lost update.
- *How does SSI detect it?* → Tracks stale snapshot reads; aborts at commit.

**Signals interviewers look for:** concrete examples beyond doctors, correct mention of phantoms, preference for constraints/serializable over hacks.

### 11. Check yourself
- [ ] Walk through the doctors example showing why both commits succeed under SI.
- [ ] Explain why a phantom prevents `FOR UPDATE` from working.
- [ ] Choose a fix for username claims, double spending, and room booking.
- [ ] Describe materializing conflicts and why it is a last resort.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Write Skew and Phantoms
- *Designing Data-Intensive Applications* — Martin Kleppmann — B › C
- *Designing Data-Intensive Applications* — Martin Kleppmann — Write Skew and Phantoms › Characterizing write skew
- *Designing Data-Intensive Applications* — Martin Kleppmann — Write Skew and Phantoms › Phantoms causing write skew
- *Designing Data-Intensive Applications* — Martin Kleppmann — Write Skew and Phantoms › Materializing conflicts
- *Designing Data-Intensive Applications* — Martin Kleppmann — Write Skew and Phantoms › More examples of write skew
- *Hadoop: The Definitive Guide* — Tom White — Spark › Note

