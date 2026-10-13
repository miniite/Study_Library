---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["PostgreSQL", "Snowflake"]
---

# Idempotency Key Tables

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An idempotency key table records "this operation (identified by key K) has already been applied", protected by a unique constraint, so retries, duplicate messages and re-run jobs change the data **once** — the database, not the application, is the arbiter.

### 1. The big picture
Distributed systems retry. Networks time out after the server committed; consumers receive a message twice; orchestrators re-run a failed task. Without protection each retry applies the effect again: a customer is charged twice, an event is counted twice, a row is inserted twice. **Idempotency** means applying an operation N times has the same effect as applying it once.

There are two broad ways to get there:
1. **Naturally idempotent writes** — "set to value" operations, `MERGE`/upsert on a business key, overwrite a partition (the "simpler overwrite" pattern in data-engineering design patterns), `DELETE` + `INSERT` for a batch.
2. **Explicit idempotency keys** — a unique key per logical operation stored in a table in the *same transaction* as the effect.

The same family covers related reliability patterns: the **outbox** (publish events exactly-once relative to the DB commit), **reject/quarantine tables**, ID generation at scale, and safe expiry/archival.

### 2. Core concepts and vocabulary
- **Idempotent operation** — repeating it yields the same state.
- **Idempotency key** — a client/job-generated unique id for one logical operation (`payment-1234`, `job=daily_load/date=2026-10-12`).
- **Unique constraint** — the mechanism that makes the second insert fail or be ignored.
- **At-least-once delivery** — messages may arrive more than once; consumers must dedupe.
- **Exactly-once effect** — at-least-once delivery + idempotent processing.
- **Outbox pattern** — write the domain change and the event row in one transaction; a separate relay publishes outbox rows.
- **Quarantine / reject table** — bad rows with a reason code kept beside the clean target.
- **Identity / sequence** — database-generated increasing numbers; **Snowflake-style ID** — time + node + sequence bits, generated without a central sequence.
- **Soft delete / anonymize** — mark or scrub rather than physically delete.

### 3. How it works, step by step
**Idempotency key table (request-level):**
1. Client sends `Idempotency-Key: K`.
2. In one transaction: `INSERT INTO idempotency(key, request_hash, status) …` — the PK/unique index guarantees one winner.
3. If the insert **succeeds**, execute the business effect, store the response, commit.
4. If it **conflicts**, read the stored record: return the saved response (if complete) or "in progress"; verify `request_hash` matches, otherwise reject (key reuse with different payload).

**Outbox:**
1. Within the business transaction insert the event row to `outbox(id, payload, published_at NULL)`.
2. A relay selects unpublished rows, publishes to the broker, then marks them published.
3. If it crashes between publish and mark, the event is published again → consumers must be idempotent. So outbox gives *no loss* and *at-least-once*; duplicates are possible.

**Batch/ETL idempotency:** key = (job, logical date). Either overwrite that partition atomically, or `DELETE WHERE batch_id = X` then `INSERT`, or upsert on the natural key.

### 4. Worked example
```sql
CREATE TABLE dbo.idempotency (
    idem_key      varchar(100) NOT NULL PRIMARY KEY,
    request_hash  binary(32)   NOT NULL,
    status        tinyint      NOT NULL,           -- 0 in progress, 1 done
    response_json nvarchar(max) NULL,
    created_at    datetime2    NOT NULL DEFAULT SYSUTCDATETIME()
);

CREATE OR ALTER PROCEDURE dbo.create_payment @key varchar(100), @hash binary(32),
                                              @customer_id int, @amount decimal(12,2)
AS
BEGIN
    SET XACT_ABORT ON;
    BEGIN TRAN;
    BEGIN TRY
        INSERT dbo.idempotency(idem_key, request_hash, status) VALUES (@key, @hash, 0);
    END TRY
    BEGIN CATCH
        IF ERROR_NUMBER() IN (2627, 2601)          -- duplicate key: already seen
        BEGIN
            ROLLBACK;
            SELECT status, response_json, CASE WHEN request_hash = @hash THEN 1 ELSE 0 END AS same_request
            FROM dbo.idempotency WHERE idem_key = @key;
            RETURN;
        END
        ELSE THROW;
    END CATCH;

    INSERT dbo.payment(customer_id, amount) VALUES (@customer_id, @amount);
    DECLARE @id int = SCOPE_IDENTITY();

    INSERT dbo.outbox(event_type, payload) VALUES ('PaymentCreated', CONCAT('{"id":', @id, '}'));
    UPDATE dbo.idempotency SET status = 1, response_json = CONCAT('{"payment_id":', @id, '}') WHERE idem_key = @key;
    COMMIT;
END;

-- Retention: expire keys older than 7 days in batches (not one huge delete)
DELETE TOP (5000) FROM dbo.idempotency WHERE created_at < DATEADD(DAY,-7,SYSUTCDATETIME());

-- Reject table: daily reject rate per rule
SELECT CAST(rejected_at AS date) AS day, rule_id, COUNT(*) AS rejects
FROM   dbo.rejects GROUP BY CAST(rejected_at AS date), rule_id ORDER BY day, rule_id;
```
Read it as: the second call with the same key hits the PK, rolls back its own attempt and returns the first call's stored answer; the payment and the outbox event commit atomically with the key.

### 5. Under the hood (optional at L4)
The guarantee rests on **unique index enforcement under concurrency**: two simultaneous inserts of the same key serialize on the index key (one blocks on the other's lock); the loser gets error 2627/2601 (or waits then fails) — so no application-level check-then-insert race exists (a `SELECT … IF NOT EXISTS … INSERT` without a lock *does* race). Identity columns: values are allocated in blocks of the engine's cache and **gaps appear** after rollbacks/restarts; they're unique, not gapless or strictly commit-ordered, and a single hot sequence can become a contention point under sharded high-rate inserts. Snowflake-style IDs (timestamp | node id | counter) are roughly time-ordered, need no coordination, but depend on clocks and node-id assignment. Cascading GDPR deletion across ~40 tables is safer as *anonymize + tombstone* in one audited procedure, or an ordered delete with a deletion-request log, than blind cascades.

### 6. Performance and design implications
- The key table is hot and small — keep it narrow, index only the PK, expire old keys by batch or partition drop.
- Archive/expire old rows by **dropping partitions** (metadata-only, minimal locking) rather than large deletes; batched `DELETE TOP (N)` is next best; moving to a cold table costs a copy but keeps history.
- Make the batch overwrite atomic: write to a staging partition and `SWITCH` it in.
- Use `MERGE`/upsert on the natural key for dimension-like data; the key is the business key itself.
- For sharded tables, use per-shard offset sequences (`START WITH shard_no, INCREMENT BY n_shards`) or Snowflake IDs; migrate by setting the new generator above the current max.
- Record `request_hash` to detect key reuse with different bodies.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PRIMARY KEY` / `UNIQUE` | Enforce one winner | Idempotency table |
| `MERGE` / upsert | Apply-or-insert | Natural-key writes |
| `INSERT … WHERE NOT EXISTS` (with `UPDLOCK, HOLDLOCK`) | Safe conditional insert | Without `MERGE` |
| `IGNORE_DUP_KEY` | Skip duplicates silently | Bulk idempotent loads |
| `SEQUENCE … INCREMENT BY n` | Per-shard IDs | Sharding |
| `ALTER TABLE … SWITCH PARTITION` | Atomic swap/drop | Overwrite / expiry |
| `SET XACT_ABORT ON` | Roll back on errors | Transactional procs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Idempotency key table: external API/messaging with retries | Operation is already a natural upsert |
| Partition overwrite: batch pipelines | Low-latency streaming of single events |
| Outbox: need DB change + event atomically | Consumers can't tolerate duplicates and can't dedupe |
| Soft delete/anonymize: audit, GDPR with referential history | Storage or legal requirement for physical erasure — do hard delete with proof |
| Snowflake IDs: multi-writer/multi-region | Gapless numbering is a legal requirement |

### 9. Common mistakes
- Check-then-insert in application code → race → rely on the unique constraint.
- Key written in a *separate* transaction from the effect → effect applied without key (or key without effect) → same transaction.
- Not storing the response → retry gets a different answer → persist it.
- Treating outbox as exactly-once → duplicate events → idempotent consumers keyed by event id.
- Idempotency table never purged → bloat → retention job.
- Assuming identity values are gapless/in commit order → wrong audits → use explicit counters if required.
- Multi-region writes without a conflict rule → divergence → single-writer per entity, CRDT, or last-writer-wins with versions.

### 10. 🎯 Interview kit
**30-second answer:** "Each logical operation carries a unique key; I insert it into a table with a unique constraint in the same transaction as the effect. A retry hits the constraint and returns the stored result instead of repeating the work. For batches I make the write idempotent by overwriting the partition or upserting on the natural key."

**Structure a longer answer:** clarify (delivery semantics, retry source, TTL) → mechanism (unique constraint, same transaction, stored response) → trade-offs (outbox duplicates, retention, partition overwrite) → scenario (payment API retried after timeout).

**Likely follow-ups:**
- *Can the outbox duplicate or lose events?* → no loss; duplicates if crash after publish — consumers dedupe.
- *Expire 90-day-old rows without hurting traffic?* → partition drop > batched delete > cold-table move.
- *GDPR delete across 40 tables?* → ordered delete or anonymize with an audit log, avoiding deep cascades.
- *Sequence contention on shards?* → per-shard offset sequences or Snowflake IDs.

**Signals interviewers look for:** reliance on constraints over app logic, transaction boundaries, honesty about delivery guarantees.

### 11. Check yourself
- [ ] Sketch the idempotency-key flow including the conflict branch.
- [ ] Explain how outbox can still duplicate events.
- [ ] Compare three expiry strategies by locking and cost.
- [ ] Explain gaps in identity columns.
- [ ] Choose an idempotent design for a nightly partition load.
<!-- explainer:end -->

## Implementations (tools)

- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)
- [Snowflake](<../../../TOOLS/Snowflake/_index.md>)

## Practice questions

- [Intermediate] [Explain how you'd design a rejects/quarantine table alongside a clean target so bad rows are captured with a rejection reason, and query the daily reject rate per rule to trend data quality.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40159_Explain_how_you_d_design_a_rejects_quarantine_table_alongsid.md>)
- [Deep] [Explain the reliable outbox pattern in SQL so domain events publish exactly-once relative to the DB transaction, and where it can still duplicate or lose events.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39919_Explain_the_reliable_outbox_pattern_in_SQL_so_domain_events_.md>)
- [Deep] [What is an auto-increment / identity column and what problems can arise with it at scale?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41086_What_is_an_auto-increment_identity_column_and_what_problems_.md>)
- [Deep] [How would you implement row-level security so tenants can only see their own data, and what are the performance and correctness pitfalls versus filtering in application code?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40315_How_would_you_implement_row-level_security_so_tenants_can_on.md>)
- [Deep] [Your team logs every LLM call including the raw response. How would you enforce a retention policy in SQL that anonymizes prompts older than 30 days without dropping aggregate metrics?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40926_Your_team_logs_every_LLM_call_including_the_raw_response_How.md>)
- [Expert] [You need a globally unique, roughly time-ordered ID across sharded tables while a shared sequence contends. How would you move to a Snowflake-style ID or per-shard offset sequences safely?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39948_You_need_a_globally_unique_roughly_time-ordered_ID_across_sh.md>)
- [Expert] [How would you architect soft multi-region writes where users write to their nearest region? Explain conflict resolution, unique-ID generation, and why a single-writer or CRDT model may be needed.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40370_How_would_you_architect_soft_multi-region_writes_where_users.md>)
- [Expert] [Your app uses a single sequence shared across sharded tables and inserts are contending on it. How would you replace it with per-shard offset sequences or a distributed ID scheme while keeping IDs globally unique and roughly time-ordered?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40418_Your_app_uses_a_single_sequence_shared_across_sharded_tables.md>)

## Related concepts

- [Deadlocks in MySQL](<../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>) *(MySQL Platform)*
- [Schema Migration Strategy](<../../Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>) *(Migrations)*
- [Online Schema Changes](<../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>) *(MySQL Platform)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Simpler Overwrite
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Solution
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Risk Management › Identification
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Orthogonal Data › A Common Problem, A Common Solution, A New Common Problem
- *Fuzzy Data Matching with SQL* — Jim Lehmer — CustomerCountByState View › B
- *Designing Data-Intensive Applications* — Martin Kleppmann — Aiming for Correctness › Operation identifiers
- *Designing Data-Intensive Applications* — Martin Kleppmann — Aiming for Correctness › The end-to-end argument
- *Designing Data-Intensive Applications* — Martin Kleppmann — Trust, but Verify › The end-to-end argument again
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — integer, but it is typically much more useful to input a timestamp using one of several supported
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n

