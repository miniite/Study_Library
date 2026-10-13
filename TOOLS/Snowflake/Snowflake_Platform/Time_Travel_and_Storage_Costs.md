---
id: "TOOLS/Snowflake/Snowflake_Platform/Time_Travel_and_Storage_Costs"
level: 4
type: "concept"
topic: "TOOLS/Snowflake/Snowflake_Platform"
tools: ["Snowflake"]
---

# Time Travel and Storage Costs

**L4 Advanced** · [↑ Snowflake Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Time Travel lets you query, clone or undrop data as it was within a retention window, Fail-safe is a further non-queryable 7-day safety net, and both are paid for as storage.

### 1. The big picture
Snowflake tables are made of immutable micro-partitions. A `DELETE`, `UPDATE`, `TRUNCATE` or `DROP` does not erase partitions; it just stops referencing them in the current table version. Snowflake keeps the old partitions for a configurable period (**Time Travel**), then for a further fixed period (**Fail-safe**), and only then frees them. That design gives cheap recovery and zero-copy cloning, and it is also why storage bills can grow even when table size looks stable.

### 2. Core concepts and vocabulary
- **Time Travel retention** — `DATA_RETENTION_TIME_IN_DAYS`; how far back you can query/clone/undrop. Default is **1 day**; Enterprise edition and above can raise permanent objects up to **90 days**. Standard edition is capped at 1.
- **Table types** — *permanent* (full retention and Fail-safe), *transient* (0–1 day, **no Fail-safe**), *temporary* (session-scoped, 0–1 day, no Fail-safe).
- **Fail-safe** — 7 days after Time Travel ends, permanent tables only, not configurable, not queryable by you; only Snowflake support can recover from it, as a last resort.
- **Zero-copy clone** — metadata copy sharing micro-partitions; only changes diverge.
- **Timestamp types** — `TIMESTAMP_NTZ` (wall-clock, no zone), `TIMESTAMP_LTZ` (stored as an instant in UTC, displayed in the session `TIMEZONE`), `TIMESTAMP_TZ` (instant plus its own offset).
- **Stream staleness** — a stream whose offset has fallen outside the source table's retention (extended up to a limit by `MAX_DATA_EXTENSION_TIME_IN_DAYS`).

### 3. How it works, step by step
1. Data changes create new micro-partitions; replaced ones are marked *removed* but kept.
2. For `retention` days they stay readable: `SELECT … AT(TIMESTAMP => …)`, `AT(OFFSET => -3600)` or `BEFORE(STATEMENT => '<query_id>')`.
3. After that they enter Fail-safe for 7 days (permanent objects), counted in storage.
4. Then they are purged.
5. `DROP` is handled the same way at object level: `UNDROP TABLE/SCHEMA/DATABASE` works within retention.

Rule of thumb for billed storage of a churny table:
`active + (changed data × retention days) + (changed data × 7 days Fail-safe) + clone-retained bytes`.

### 4. Worked example
```sql
-- accidental delete at ~10:05, noticed later
SELECT COUNT(*) FROM sales.orders AT(TIMESTAMP => '2026-10-12 10:00:00'::TIMESTAMP_LTZ);

-- recover into a new table (clone cannot be pointed at "now minus delete" in place)
CREATE TABLE sales.orders_restored CLONE sales.orders
  BEFORE(STATEMENT => '01b2c3d4-0000-aaaa-0000-000000000000');

-- or INSERT the missing rows back
INSERT INTO sales.orders
SELECT * FROM sales.orders AT(OFFSET => -3600)
EXCEPT SELECT * FROM sales.orders;

-- reproduce a consumer's bug as of yesterday 14:30
CREATE TABLE dbg.orders CLONE sales.orders AT(TIMESTAMP => '2026-10-11 14:30:00'::TIMESTAMP_LTZ);

-- where is the storage going?
SELECT table_name, active_bytes, time_travel_bytes, failsafe_bytes, retained_for_clone_bytes
FROM snowflake.account_usage.table_storage_metrics
ORDER BY time_travel_bytes + failsafe_bytes DESC LIMIT 20;
```
If `time_travel_bytes` and `failsafe_bytes` dwarf `active_bytes`, the table churns heavily; if `retained_for_clone_bytes` is big, an old clone is pinning partitions.

### 5. Under the hood
Time Travel is a property of the *micro-partition metadata* history, not a separate log. That is why it applies per object and why cloning at a past point is a metadata operation. Retention set at account, database, schema or table level is inherited downward unless overridden. A table that is truncated and reloaded daily moves its *entire* contents into Time Travel then Fail-safe each day, so a 100 GB table with 30-day retention carries roughly 100 GB × (30 + 7) of hidden storage in steady state.

### 6. Performance and design implications
- Retention costs storage, not query speed on current data.
- Match retention to the recovery SLA: if mistakes are noticed within two days, 3–7 days may be plenty; reserve 30–90 for critical, low-churn tables.
- Staging/scratch tables that are fully rewritten should be **transient** with retention 0 or 1; recoverability comes from the source.
- Keep per-table `DATA_RETENTION_TIME_IN_DAYS` explicit for important tables so a schema-level change doesn't surprise you.
- Rebuilding is cheaper than keeping history for data you can regenerate.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER TABLE t SET DATA_RETENTION_TIME_IN_DAYS = n` | change retention | tune cost vs recovery |
| `AT / BEFORE (TIMESTAMP, OFFSET, STATEMENT)` | historic reads | recovery, audit |
| `UNDROP TABLE/SCHEMA/DATABASE` | restore dropped object | accidents |
| `CREATE TRANSIENT TABLE` | no Fail-safe, max 1 day | staging, scratch |
| `TABLE_STORAGE_METRICS` | bytes by category | find storage hogs |
| `ALTER SESSION SET TIMEZONE` | LTZ display and parsing zone | consistent timestamps |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Long retention on low-churn, high-value tables | high-churn staging tables |
| Transient tables for reloadable data | data that is the only copy |
| `TIMESTAMP_NTZ` for business wall-clock times | you need to compare instants across zones (use LTZ/TZ) |
| `TIMESTAMP_TZ` when the original offset matters | you want everything displayed uniformly |

### 9. Common mistakes
- Believing the default retention is 90 days → it is 1 day → check and set it deliberately.
- Describing Fail-safe as something you query or configure → it is Snowflake-operated recovery only.
- Mixing up types: Time Travel recovers *after the fact*; it is not a pre-operation guard.
- Ignoring session `TIMEZONE` when parsing strings into LTZ → values shift silently → set the zone, load into explicit types, and validate with row-level spot checks and min/max timestamp reconciliation against the source.
- Letting a stream go stale (not consumed within retention) → CDC gap → size retention to the longest expected consumer outage, monitor `SYSTEM$STREAM_GET_TABLE_TIMESTAMP`/staleness, and recreate plus backfill if lost.

### 10. 🎯 Interview kit
**30-second answer:** Time Travel retains old micro-partitions for a configurable window (default 1 day, up to 90 on Enterprise) so I can query, clone or undrop; Fail-safe adds 7 non-queryable days for permanent tables. Both bill as storage, so I set retention per table by recovery SLA and churn.

**Structure a longer answer:** clarify (edition, table type) → mechanism (immutable partitions) → cost model → scenario (churny staging table → transient, retention 0–1).

**Likely follow-ups:**
- *Fail-safe vs Time Travel?* → user-driven and queryable vs Snowflake-only last resort.
- *Why is storage climbing with stable table size?* → `TABLE_STORAGE_METRICS`: Time Travel, Fail-safe or clone-retained bytes.
- *Reproduce yesterday's bug?* → clone `AT(TIMESTAMP)`.
- *NTZ vs LTZ vs TZ?* → wall clock vs instant shown in session zone vs instant with stored offset.

**Signals interviewers look for:** correct defaults, cost awareness, knowing transient tables, using metrics rather than guessing.

### 11. Check yourself
- [ ] Compute hidden storage for a 200 GB daily-reloaded table with retention 14 (permanent).
- [ ] Write the statement to recover rows deleted an hour ago.
- [ ] Explain when a stream turns stale and how to recover.
- [ ] Choose a timestamp type for event time from mobile clients worldwide.
<!-- explainer:end -->

## Implementations (tools)

- [Snowflake](<../_index.md>)

## Practice questions

- [Basic] [What is the default Time Travel retention period and how can it be changed?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39457_What_is_the_default_Time_Travel_retention_period_and_how_can.md>)
- [Intermediate] [What is Time Travel in Snowflake and how would you use it to recover accidentally deleted data?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39456_What_is_Time_Travel_in_Snowflake_and_how_would_you_use_it_to.md>)
- [Intermediate] [What is Fail-safe in Snowflake and how does it differ from Time Travel?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39458_What_is_Fail-safe_in_Snowflake_and_how_does_it_differ_from_T.md>)
- [Intermediate] [How do you handle date and timestamp time zones in Snowflake?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39538_How_do_you_handle_date_and_timestamp_time_zones_in_Snowflake.md>)
- [Intermediate] [What are the TIMESTAMP_NTZ, TIMESTAMP_LTZ, and TIMESTAMP_TZ types and when do you use each?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39539_What_are_the_TIMESTAMP_NTZ_TIMESTAMP_LTZ_and_TIMESTAMP_TZ_ty.md>)
- [Intermediate] [How do you decide the Time Travel retention for a table given both cost pressure and the SLA for accidental-delete recovery?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39590_How_do_you_decide_the_Time_Travel_retention_for_a_table_give.md>)
- [Intermediate] [How do you use CLONE plus Time Travel together to reproduce a bug that a consumer hit at a specific timestamp yesterday?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39605_How_do_you_use_CLONE_plus_Time_Travel_together_to_reproduce_.md>)
- [Intermediate] [A COPY load succeeds but numbers are subtly wrong due to timezone/date parsing; how would you catch this class of bug systematically?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39626_A_COPY_load_succeeds_but_numbers_are_subtly_wrong_due_to_tim.md>)

## Related concepts

- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Transaction vs Snapshot Facts](<../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts.md>) *(Dimensional Modeling)*
- [Lakehouse Fundamentals](<../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*

