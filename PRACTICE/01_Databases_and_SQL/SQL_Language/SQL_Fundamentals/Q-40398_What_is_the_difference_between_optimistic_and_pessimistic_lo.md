---
type: "practice"
difficulty: "Deep"
---

# What is the difference between optimistic and pessimistic locking?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ SKIP LOCKED Job Queues](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: The interviewer wants me to explain two concurrency-control strategies for protecting shared data when multiple transactions (or app threads) touch the same rows — and, more importantly, when to pick which. This is a "deep understanding" question, so a name-the-definition answer is not enough. They're probing whether I understand: (1) the mechanism each uses, (2) the failure/contention trade-offs, (3) how each maps to real database features (row locks, SELECT ... FOR UPDATE, version columns), and (4) judgment about which to use for a given workload (e.g. inventory decrement vs. profile edit).

Key assumption / clarifying framing: Both approaches solve the lost-update problem — two transactions read a value, both compute a new value, and one overwrites the other silently. The difference is when they check for conflict: before the work (pessimistic) or at commit time (optimistic). I'll assume a relational DB (Postgres/MySQL InnoDB), because the interviewer said SQL, but I'll note the pattern is identical in app-level and NoSQL systems.

Plan: define the lost-update problem → define each strategy from first principles → show the concrete SQL/mechanism for each → compare on contention, latency, deadlocks, scalability, correctness → give a decision rule with real examples (payments, inventory, seat booking) → summary and common mistakes.

The problem both solve: lost update

#### Cons
ider two users buying the last item. Stock = 1.

CODE
T1: read stock = 1        T2: read stock = 1
T1: 1 >= 1, ok            T2: 1 >= 1, ok
T1: write stock = 0       T2: write stock = 0   <-- both succeed, 2 items sold, 1 in stock

Both transactions read the same value, neither saw the other's write, and one update is lost. Concurrency control is how we stop this. Optimistic and pessimistic locking are the two philosophies.

Pessimistic locking — "assume conflict, lock first"

Idea: Before you read-modify-write a row, acquire a lock on it so no one else can touch it until you commit. Conflict is prevented by making other transactions wait.

In SQL this is SELECT ... FOR UPDATE inside a transaction:

SQL
BEGIN;
SELECT stock FROM products WHERE id = 42 FOR UPDATE;  -- row is now locked
-- if stock >= 1:
UPDATE products SET stock = stock - 1 WHERE id = 42;
COMMIT;  -- lock released

Now T2's SELECT ... FOR UPDATE on the same row blocks until T1 commits. When it wakes, it reads the fresh value (0) and correctly refuses. The lock serializes access to that row.

Guarantees: No lost update by construction; the read reflects the latest committed value.
Cost: Contention. While the lock is held, others wait — throughput on hot rows drops. Locks held across network/user "think time" are disastrous (never hold a DB lock while waiting for a user). Risk of deadlocks if two transactions lock rows in different orders (the DB will detect and kill one). Doesn't scale well across distributed nodes because a lock is shared state.
Optimistic locking — "assume no conflict, verify at commit"

Idea: Take no lock while working. Read the row, remember its version, do your logic, and at write time verify the row hasn't changed since you read it. If it did, you lost the race — abort and retry. Conflict is detected, not prevented.

Two common mechanisms:

(a) Version column (most explicit):

SQL
-- Read
SELECT stock, version FROM products WHERE id = 42;   -- e.g. stock=1, version=7
-- ... application computes new stock=0 ...
-- Write only if version is still 7:
UPDATE products
SET stock = 0, version = version + 1
WHERE id = 42 AND version = 7;
-- If rows_affected = 0 -> someone else updated it first -> conflict -> retry

If T2 committed between our read and write, version is now 8, our WHERE version = 7 matches 0 rows, and we know to re-read and retry. (b) The same effect can be had with a timestamp/updated_at, or by putting the old value directly in the WHERE (... AND stock = 1) — a compare-and-swap.

Guarantees: No lost update, because the conditional update only applies if the world hasn't changed.
Cost: Wasted work on conflict (you re-do the transaction), and you must write retry logic. Under high contention on one row, retries pile up and effective throughput collapses (livelock-ish). Under low contention it's nearly free — no locks, no waiting.
Head-to-head comparison

The core trade-off: pessimistic pays a fixed cost up front (waiting) to guarantee first-try success; optimistic pays nothing up front but a retry cost on conflict. So the right choice is driven by how often two transactions actually collide.

Low contention (edits to a user's own profile, most CRUD): optimistic wins — no lock overhead, conflicts are rare so retries almost never fire.
High contention on a hot row (the single "last ticket", a counter everyone increments): pessimistic often wins, or you redesign the schema (sharded counters, reserving rows) to remove the hotspot. Pure optimistic here degenerates into a retry storm.
Long transactions / think-time: never hold a pessimistic lock across a user round-trip. Optimistic is the natural fit for stateless web requests and "edit form then save 2 minutes later" flows.
Deadlocks: a pessimistic-locking risk (mitigate with consistent lock ordering, short transactions, NOWAIT/SKIP LOCKED). Optimistic has no deadlocks — worst case is repeated aborts.
Distributed / stateless systems: optimistic (version/CAS) travels well; a version number is just data. Pessimistic locks are DB-node-local shared state and are painful to hold across services.
Decision rule (what I'd say to close)

"Default to optimistic for typical web CRUD where two writers rarely hit the same row — it's lock-free, scales, and handles long user think-time. Switch to pessimistic (SELECT ... FOR UPDATE) when contention on specific rows is genuinely high and retries would thrash, or when I need to lock a set of rows atomically for a short critical section. For extreme hotspots (inventory of a flash sale, seat booking), I'd combine pessimistic locking with schema tricks like SELECT ... FOR UPDATE SKIP LOCKED to grab any free seat without everyone queueing on the same row."

Note these are not the only tools: the DB's isolation level matters too. SERIALIZABLE gives you optimistic-style conflict detection (serialization failures you retry) automatically; REPEATABLE READ in Postgres also surfaces update conflicts. Application-level version columns are just making that control explicit and portable.

Summary — what a strong answer emphasizes

Pessimistic = prevent conflicts by locking and making others wait (FOR UPDATE); best when collisions are frequent and transactions short. Optimistic = detect conflicts at commit via a version/timestamp/CAS and retry; best when collisions are rare, transactions are long or stateless, or the system is distributed. Both eliminate lost updates; they differ in where they pay the cost — waiting vs. retrying — and the interviewer wants to see you reason from the contention profile of the specific workload, plus awareness of deadlocks, retry storms, and isolation levels.

#### Common mistakes
to avoid: (1) Holding a pessimistic lock across user think-time — kills concurrency. (2) Doing optimistic "read then update" without the WHERE version = ? guard — that's just a plain update and reintroduces lost update. (3) Forgetting retry logic with optimistic locking. (4) Ignoring that a plain UPDATE ... SET stock = stock - 1 (atomic single-statement) already avoids lost update for that specific case — you don't always need either strategy if you can express the change as one atomic statement.

**Comparison:**

```
Aspect	Pessimistic locking	Optimistic locking
Conflict handling	Prevent (lock, others wait)	Detect at commit, then retry
Mechanism	SELECT ... FOR UPDATE, row/table locks	version / timestamp column, compare-and-swap
Cost model	Pay up front by waiting	Free unless conflict; retry cost on conflict
Best when	High contention, short txns	Low contention, long/stateless txns
Deadlocks	Possible (need lock ordering)	None (only repeated aborts)
Throughput on hot row	Serialized but predictable	Degrades to retry storm
Distributed friendliness	Poor (lock = shared node state)	Good (version travels as data)
Lost update prevented?	Yes	Yes (via WHERE version = ?)
```

**Code:**

```

-- PESSIMISTIC: lock the row for the duration of the transaction BEGIN; SELECT stock FROM products WHERE id = 42 FOR UPDATE; -- others block here UPDATE products SET stock = stock - 1 WHERE id = 42 AND stock > 0; COMMIT; -- lock released

-- OPTIMISTIC: version column + compare-and-swap, retry on conflict -- 1) read SELECT stock, version FROM products WHERE id = 42; -- stock=1, version=7 -- 2) conditional write UPDATE products SET stock = stock - 1, version = version + 1 WHERE id = 42 AND version = 7 AND stock > 0; -- 3) if @@ROWCOUNT / rows_affected == 0 -> conflict -> re-read and retry

CODE
Copy
```pseudocode
// Optimistic retry loop (application side)
for attempt in 1..MAX_RETRIES:
    (stock, version) = db.read(id)
    if stock < 1: return SOLD_OUT
    affected = db.update("... WHERE id=? AND version=?", newStock, version+1, id, version)
    if affected == 1: return SUCCESS       // won the race
    // else someone updated first -> loop and re-read
return CONFLICT_GIVE_UP
```

**Example input / output:**

```

Scenario: stock = 1, two concurrent buyers.

PESSIMISTIC trace:

CODE
Copy
T1 SELECT ... FOR UPDATE  -> locks row, reads stock=1
T2 SELECT ... FOR UPDATE  -> BLOCKS (waits for T1)
T1 UPDATE stock=0; COMMIT -> lock released
T2 unblocks, reads stock=0 -> refuses purchase   ==> exactly 1 sold  (correct)

OPTIMISTIC trace:

CODE
Copy
T1 read (stock=1, ver=7)      T2 read (stock=1, ver=7)
T1 UPDATE ... WHERE ver=7 -> affected=1 (ver now 8, stock 0)  -> SUCCESS
T2 UPDATE ... WHERE ver=7 -> affected=0 (ver is 8, not 7)     -> CONFLICT
T2 re-reads (stock=0) -> refuses purchase        ==> exactly 1 sold  (correct)
```

**Data:**

```sql
-- Table shape enabling optimistic locking (note the version column)
CREATE TABLE products (
  id       BIGINT PRIMARY KEY,
  stock    INT NOT NULL CHECK (stock >= 0),
  version  INT NOT NULL DEFAULT 0     -- bumped on every update; the conflict detector
);
```

**Flow:**

1. High / short txn
2. Low / long or stateless txn
3. Yes
4. No
5. Two txns want same row
6. Expected contention?
7. Pessimistic: SELECT ... FORUPDATE
8. Optimistic: read version
9. Others block until commit
10. Update + COMMIT -> lockfree
11. UPDATE ... WHERE version =read_version
12. rows_affected == 1?
13. Success
14. Conflict: re-read and retry

