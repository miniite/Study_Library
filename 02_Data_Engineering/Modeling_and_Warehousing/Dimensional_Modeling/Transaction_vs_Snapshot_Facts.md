---
id: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling"
tools: []
---

# Transaction vs Snapshot Facts

**L4 Advanced** · [↑ Dimensional Modeling](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A transaction fact records each event as it happens, a periodic snapshot records state at regular intervals, and an accumulating snapshot tracks one process instance through its milestones in a single updated row.

### 1. The big picture
Kimball describes three fundamental fact table types, each answering a different kind of question. **Transaction facts** answer "what happened?" — one row per event, never updated. **Periodic snapshot facts** answer "what was the state at the end of each period?" — one row per entity per period, even if nothing happened. **Accumulating snapshot facts** answer "where is each item in its pipeline, and how long did each step take?" — one row per process instance, updated as milestones occur.

Choosing the wrong type causes either impossible questions (no state without replaying every event) or silent wrong answers (summing balances across days). In practice a mature warehouse has several types for the same business process: orders as transactions, inventory as a periodic snapshot, order fulfilment as an accumulating snapshot.

### 2. Core concepts and vocabulary
- **Transaction fact table** — one row per measurement event at the atomic grain; rows are inserted, almost never changed. Sparse: no event, no row.
- **Periodic snapshot** — one row per entity per period (account per day, SKU per store per week) capturing state such as balance or stock on hand; dense: rows exist even with no activity.
- **Accumulating snapshot** — one row per lifecycle instance (an order, a claim), with a date key per milestone and lag measures between them; the row is **updated** as the process advances.
- **Semi-additive measure** — can be summed across entities but not over time (balances, stock levels).
- **Milestone date key** — one role-played foreign key to the date dimension per stage (ordered, packed, shipped, delivered).
- **Lag measure** — days between two milestones, e.g. `order_to_ship_days`.
- **Factless fact** — an event row with no numeric measure (attendance), a variant of transaction facts.
- **Point-in-time join** — joining a fact to the dimension version valid at the event date (SCD Type 2 range).

### 3. How it works, step by step
**Transaction fact**
1. Capture each event from the source (CDC or batch extract).
2. Look up surrogate keys for every dimension as of the event date.
3. Insert the row with additive measures (amount, quantity). Corrections are posted as new rows (reversal + restatement) or via update with audit — rows are not normally rewritten.

**Periodic snapshot**
1. At each period end (or daily), read the state of every entity.
2. Insert one row per entity with measures such as `ending_balance`, `units_on_hand`, plus activity measures if useful (`deposits_in_period`).
3. Query with the right aggregation: `SUM` across accounts for a day, `AVG` or last-value across days.

**Accumulating snapshot**
1. When the process starts, insert a row with the start date key filled and later milestone keys pointing to a "not yet happened" placeholder date row.
2. As each milestone occurs, `UPDATE` that row: set the milestone date key and compute the lag measures.
3. Rows reach a final state (closed) and stop changing.

| Property | Transaction | Periodic snapshot | Accumulating snapshot |
|---|---|---|---|
| Row means | one event | entity state per period | one process instance |
| Rows added | when events occur | every period, for every entity | once per instance |
| Updates | rare | never | yes, several |
| Density | sparse | dense | one per instance |
| Time keys | one date (event) | the snapshot date | several milestone dates |
| Typical measures | amount, qty | balance, stock level | lags, counts of steps |

### 4. Worked example
Banking: deposits/withdrawals as transactions, end-of-day balance as snapshot.

```sql
CREATE TABLE fact_account_txn (
    date_key int, account_key int, txn_type_key int,
    txn_id bigint, amount decimal(14,2)
);

CREATE TABLE fact_account_daily_balance (
    date_key int, account_key int,
    ending_balance decimal(16,2),       -- semi-additive
    txn_count int, net_change decimal(14,2),
    PRIMARY KEY (date_key, account_key)
);

-- total bank deposits balance on one day: OK to SUM across accounts
SELECT SUM(ending_balance) FROM fact_account_daily_balance WHERE date_key = 20260630;

-- average balance per account over the month: AVG across days, not SUM
SELECT account_key, AVG(ending_balance)
FROM fact_account_daily_balance
WHERE date_key BETWEEN 20260601 AND 20260630
GROUP BY account_key;
```

Order fulfilment as an accumulating snapshot:

```sql
CREATE TABLE fact_order_fulfilment (
    order_number varchar(20) PRIMARY KEY,
    customer_key int, product_key int,
    order_date_key int, pick_date_key int, ship_date_key int, deliver_date_key int,
    order_qty int, order_to_ship_days int, ship_to_deliver_days int
);

-- milestone arrives
UPDATE f
SET ship_date_key = d.date_key,
    order_to_ship_days = DATEDIFF(day, od.full_date, d.full_date)
FROM fact_order_fulfilment f
JOIN stg_ship s ON s.order_number = f.order_number
JOIN dim_date d ON d.full_date = s.ship_date
JOIN dim_date od ON od.date_key = f.order_date_key;
```

Reading it: unfinished orders have `deliver_date_key` pointing to the placeholder ("Not delivered"), so `WHERE deliver_date_key = -1` lists the open pipeline, and `AVG(order_to_ship_days)` is the shipping SLA.

**Currency:** keep the transaction currency and amount on the fact, plus a reporting-currency amount computed with the rate at load time and the `rate_key` (or rate) retained, so conversion is reproducible and restating with new rates does not require recomputing from scratch.

### 5. Under the hood
A periodic snapshot is, conceptually, a materialised running total of the transaction stream, evaluated at fixed times. It costs storage proportional to entities × periods (even idle ones), which is why granularity decisions (daily vs weekly; per bin vs per SKU) matter. A common compromise: **snapshot only on change** plus a date range (`valid_from`, `valid_to`) — the same shape as SCD Type 2, and queryable by range join. This is smaller but needs range logic for "state on day X".

An accumulating snapshot is the only fact type that is mutated in place, so it needs idempotent upsert logic (`MERGE`) keyed on the process instance, handling out-of-order and late milestones. In columnar/immutable stores (Parquet, Delta), frequent row updates cost file rewrites; teams often keep an append-only **event-log fact** and derive the accumulating view with a window function or a periodic rebuild.

### 6. Performance and design implications
- Transactions: biggest volumes; partition by date, append-only loads are cheap and parallel.
- Snapshots: predictable linear growth (entities × periods); reduce by coarser period, dropping empty entities, or snapshot-on-change.
- Accumulating: small and fast to query (one row per instance), but updates need a good key index (or `MERGE` on the instance key) and are costly on append-only formats.
- Wrong aggregation of semi-additive measures is the dominant correctness bug — mark such measures in the semantic layer so BI tools use last-value/average.
- Time zones: store both a UTC timestamp and the local date key so local-day and UTC reporting both work; do not derive local date from UTC at query time.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE … WHEN MATCHED THEN UPDATE` | upserts milestone data into an accumulating row | milestone arrives |
| `LAST_VALUE() OVER (… ORDER BY date)` / `MAX` by date | picks the end-of-period state | aggregating snapshots over time |
| `AVG(balance)` over days | average level | semi-additive measure across time |
| `Partition by date_key` | prunes scans | large transaction/snapshot facts |
| `LAG()` / `DATEDIFF` | milestone lag measures | accumulating snapshot |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Transaction fact: events are the business question; need full detail | You need "state on any day" frequently — replaying events is slow |
| Periodic snapshot: state-based questions (balances, stock) | Entities are huge in number and mostly idle (storage blow-up) |
| Accumulating snapshot: bounded process with known milestones and SLA analysis | Steps are unpredictable or repeating; use an event-log fact instead |
| Snapshot-on-change | Consumers cannot write range joins |

### 9. Common mistakes
- Summing balances across days → inflated numbers → use average/last value for the time dimension.
- Using only transactions for inventory → must replay history to know stock → add a snapshot.
- Updating transaction rows to fix errors silently → reports already signed off change → post reversal rows or version the data.
- Forcing an accumulating snapshot for processes that loop (re-open, re-ship) → overwritten history → use an event log.
- Filling milestone keys with NULL → broken joins to date dimension → use a placeholder "unknown/not occurred" date row.

### 10. 🎯 Interview kit
**30-second answer:** Transaction facts are one row per event; periodic snapshots are one row per entity per period for state like balances; accumulating snapshots are one row per process instance updated as milestones occur. I pick by the question: what happened, what was the state, or where is it in the pipeline.

**Structure a longer answer:** define the three → give a banking or order example for each → explain additivity (semi-additive balances) → discuss storage/update cost → mention hybrids (event log, snapshot-on-change).

**Likely follow-ups:**
- *Why not derive balances from transactions only?* → possible but expensive for point-in-time queries and fragile if history is incomplete; snapshots make it a lookup.
- *How big does a snapshot get?* → entities × periods; control with grain choice and snapshot-on-change.
- *How do you do a point-in-time-correct join?* → SCD2 dimension with `valid_from/valid_to`; join on key and event date within the range, indexed on (natural key, valid_from).
- *Multi-currency?* → store original amount and currency, plus converted amount and rate used.

**Signals interviewers look for:** correct semi-additivity, awareness that accumulating snapshots are updated, and a pragmatic hybrid where one type is insufficient.

### 11. Check yourself
- [ ] Explain why a daily balance cannot be summed across a month.
- [ ] Design the columns of an accumulating snapshot for loan application processing.
- [ ] Name when you would prefer an event-log fact to an accumulating snapshot.
- [ ] Write a query returning the average daily balance per account for a month.
- [ ] Describe how snapshot-on-change reduces storage and what it complicates.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a transaction fact table? Give an everyday example.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12498_What_is_a_transaction_fact_table_Give_an_everyday_example.md>)
- [Intermediate] [Give an example where a daily snapshot fact table makes more sense than a transaction one.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12500_Give_an_example_where_a_daily_snapshot_fact_table_makes_more.md>)
- [Deep] [What is a periodic snapshot fact table and when is it useful, for example daily account balances?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12176_What_is_a_periodic_snapshot_fact_table_and_when_is_it_useful.md>)
- [Deep] [What is the difference between a snapshot of current state and a full transaction history in fact design?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12239_What_is_the_difference_between_a_snapshot_of_current_state_a.md>)
- [Deep] [How do you handle time zones in a global fact table so both local-time and UTC reporting are correct, and where do you store the offset?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12339_How_do_you_handle_time_zones_in_a_global_fact_table_so_both_.md>)
- [Deep] [How do you handle currency conversion in a payments fact so reports slice by both transaction currency and a single reporting currency, and where do you store the rate?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12573_How_do_you_handle_currency_conversion_in_a_payments_fact_so_.md>)
- [Deep] [How would you build a point-in-time-correct join so a report shows facts against the dimension as it looked at the event date, and how do you index it to stay fast?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12613_How_would_you_build_a_point-in-time-correct_join_so_a_report.md>)
- [Expert] [How do you model a payments transaction fact so it supports both real-time fraud scoring and end-of-day reconciliation without two separate schemas?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12307_How_do_you_model_a_payments_transaction_fact_so_it_supports_.md>)

## Related concepts

- [Fact Dimension Integrity](<../Data_Warehousing/Fact_Dimension_Integrity.md>) *(Data Warehousing)*
- [Joining Type 2 Dimensions](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>) *(SQL Fundamentals)*
- [Denormalization Trade-offs and Patterns](<../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>) *(Normalization)*

