---
id: "02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Incremental_Sync_and_Connector_Migration"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion"
tools: []
---

# Incremental Sync and Connector Migration

**L4 Advanced** · [↑ Data Ingestion](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Managed connectors (Fivetran, Airbyte and similar) sync sources to a destination using stored state; running them well means choosing sync modes and frequencies deliberately, treating connector configuration as code, and making incremental syncs trustworthy.

### 1. The big picture
Most teams ingest from dozens of SaaS apps and databases. Writing custom extractors for each is expensive, so they use connector platforms: Fivetran (managed, usage-priced), Airbyte (open source or cloud), Meltano/Singer taps, and cloud-native equivalents. Each connector knows how to authenticate to a source, discover its schema, extract data, and load it to a destination with state tracking.

The engineering work shifts from writing extractors to **operating** them: deciding how each stream syncs, how often, how it is orchestrated with downstream transformations, how changes to configuration are promoted safely, and how you know data is complete.

### 2. Core concepts and vocabulary
- **Connection** — a configured pairing of source, destination and selected streams/tables.
- **Sync** — one run that extracts and loads data; triggered by schedule, API call or event.
- **Full refresh** — re-extracts everything in the stream. Modes: overwrite (replace) or append.
- **Incremental sync** — extracts only new/changed records using a **cursor field** (e.g. `updated_at`) and stored state.
- **Cursor field** — the column used to decide what is new.
- **Primary key** — used to deduplicate and upsert.
- **Sync mode** — combination such as *incremental append* (adds every extracted version of a record), *incremental dedupe (upsert)* (keeps latest per key), *full refresh overwrite*, *full refresh append*.
- **State** — the stored cursor/position per stream, persisted by the platform so the next sync resumes.
- **Monthly Active Rows (MAR)** — a usage-pricing unit: the count of distinct rows inserted/updated by connectors in a month, so frequently changing rows cost more but unchanged rows don't.
- **Reconciliation** — comparing source and destination counts/aggregates to check completeness.
- **Freshness SLA** — maximum acceptable age of data per source.

### 3. How it works, step by step
1. **Discover** the schema; the user selects streams and columns.
2. **Initial sync** loads historical data (a full read), writing to the destination and recording state.
3. **Subsequent syncs** read the cursor from state, request `cursor > state` from the source, write results, update state on success.
4. **Dedupe/upsert** (if configured) happens in the destination, using primary key plus cursor, often via a raw table and a final typed table.
5. **Downstream triggers**: after a successful sync, transformations (dbt) run, either on a schedule that assumes syncs finished or through an orchestrator that triggers them when the sync completes.
6. **Monitoring** records sync status, rows synced, duration and failures; alerts fire on failures or stale data.

### 4. Worked example
60 connectors, a fixed budget, and a requirement of near-real-time data for one executive dashboard.

Tiering approach:
| Tier | Sources | Frequency | Rationale |
|---|---|---|---|
| 1 | Orders DB, payments | 5–15 min | revenue dashboards, operations |
| 2 | CRM, support | hourly | team workflows |
| 3 | Marketing, finance exports | daily | batch reporting |
| 4 | Reference/static | weekly or manual | rarely changes |

Orchestration with Airflow so dbt only runs on fresh data:

```python
sync = AirbyteTriggerSyncOperator(task_id="sync_orders", connection_id=ORDERS_CONN, asynchronous=False)
dbt = BashOperator(task_id="dbt_orders", bash_command="dbt build -s tag:orders")
sync >> dbt
```

Configuration as code: connections are defined in Terraform or the platform's API/YAML, committed to Git, and promoted dev → staging → prod through CI, rather than clicking in a UI.

**Fixing an untrustworthy incremental.** The source application updates rows in place without touching `updated_at`. Incremental sync will never see the change. Options: switch that stream to CDC (log-based); use a different cursor such as a database row version or system-maintained column; run periodic full refresh with dedupe for that table; or add a derived "last modified" via a trigger. Add reconciliation to prove it works.

### 5. Under the hood
- **State storage.** The platform stores per-stream cursors; Fivetran keeps it internally, Airbyte stores it in its database (and exposes it via API). Migrating a stream from full refresh to incremental should *seed* state with a safe starting cursor (typically the max cursor already in the destination minus a lookback) so the next sync doesn't re-pull everything. Without state migration you re-sync the entire table.
- **Append vs dedupe.** Append keeps every version, producing a history you can use but need to collapse; dedupe maintains one row per key, requiring a reliable primary key and cursor. Missing or non-unique keys cause duplicates or lost updates.
- **Deletes.** API and query-based incremental syncs usually cannot see deletions; connectors may provide soft-delete flags, separate "deleted records" endpoints, or only log-based CDC captures them.
- **API limits.** SaaS APIs return HTTP 429 when rate limited; syncs slow dramatically. Mitigation: reduce selected streams/columns, stagger connectors, use bulk/export endpoints, and increase intervals so each sync handles a similar total volume without hitting limits. Never "retry hard" without backoff.
- **Pricing mechanics.** In MAR-based pricing, cost relates to the number of *distinct changed rows*, not the number of syncs, so syncing more often doesn't automatically cost more but re-syncs and noisy updated columns do. Exclude volatile columns or tables when they inflate active rows.
- **Schema drift.** New source columns may appear automatically; policies decide whether to propagate, block or alert.

### 6. Performance and design implications
- Sync frequency is a trade between freshness, source/API load, destination load and cost; decide per connection from the downstream SLA.
- Scheduling syncs independently of dbt leads to transforming stale data. Event-driven orchestration (trigger dbt when sync completes) removes this at the cost of additional wiring.
- Monitor freshness per source with an SLA (e.g. `max(_synced_at)` or max event time), alert the owning team, and track sync duration trends.
- Before promoting a new connector: test in a non-prod destination, compare counts and aggregates, run a canary period, and have a rollback.
- Keep a documented rollback plan and rehearse it.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Sync schedule (cron/interval) | When syncs run | Match to SLA tiers |
| Incremental cursor field + primary key | Defines what's new and how to dedupe | Every incremental stream |
| Sync mode (append / dedupe / overwrite) | Controls destination semantics | History vs current-state needs |
| Connector API/Terraform provider | Manage connections as code | Promotion across environments |
| Table/column selection and hashing | Limits data and protects PII | Cost and compliance |
| `_synced_at` / `_airbyte_extracted_at` metadata columns | Records load time | Freshness monitoring |
| Reconciliation queries | Compare source/target aggregates | After migrations and periodically |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Incremental dedupe for mutable entities with a trustworthy cursor and key | Cursor can be skipped by in-place updates or keys are not unique |
| Incremental append for audit/history | Consumers expect current-state tables without a dedupe layer |
| Full refresh for small reference tables | Large tables; costly and noisy on MAR pricing |
| Scheduled syncs: simple, predictable | Strict freshness or dependent transformations |
| Event-driven triggers (sensors, API) | Added complexity not justified for daily batch |
| Managed SaaS connectors: speed | Exotic sources, strict cost control, custom semantics (build or open source) |

### 9. Common mistakes
- Relying on timestamps alone → in-place updates missed → CDC, row versions, or periodic full refresh.
- Switching modes without migrating state → full re-sync, cost spike, long window → seed state first.
- Syncing everything every 5 minutes "because we can" → cost and API throttling → tier frequencies.
- Treating UI changes as deployment → drift between environments → configuration as code.
- Skipping reconciliation → silent gaps → scheduled count/aggregate checks per stream.
- Running dbt on a fixed clock after syncs → stale or partial data transformed → trigger on completion.

### 10. 🎯 Interview kit
**30-second answer:** A managed connector tracks per-stream state and syncs either full refresh or incrementally using a cursor and primary key. I tier sync frequency by business SLA and cost, orchestrate downstream dbt runs on sync completion, manage connection config as code, and reconcile source versus destination to trust incrementals.

**Structure a longer answer:** modes and state → frequency decisions → orchestration → promotion and rollback → failure handling (429s, drift, missed updates) → monitoring and cost.

**Likely follow-ups:**
- *Append vs dedupe mode?* → history vs one row per key; dedupe requires a reliable key and cursor.
- *Migrate full-refresh to incremental without re-sync?* → seed state from destination max cursor with lookback, validate, switch.
- *6-hour syncs due to 429s?* → reduce scope, stagger, bulk endpoints, adjust frequency, backoff; don't drop data.
- *Monitor freshness for dozens of connectors?* → per-source SLA table fed from sync metadata, alert to the owning team.

**Signals interviewers look for:** operations mindset, awareness of cost models, and trust-building (reconciliation).

### 11. Check yourself
- [ ] Explain what state a connector stores and what happens if it is lost.
- [ ] Choose a sync mode for orders, a currency lookup table, and an audit log.
- [ ] Design an orchestration where dbt runs only after fresh landing.
- [ ] Describe a safe migration from full refresh to incremental.
- [ ] List two reasons an incremental sync misses updates and fixes.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between full refresh and incremental sync in data ingestion?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11646_What_is_the_difference_between_full_refresh_and_incremental_.md>)
- [Intermediate] [What is Monthly Active Rows (MAR) in Fivetran's pricing model?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11674_What_is_Monthly_Active_Rows_MAR_in_Fivetran_s_pricing_model.md>)
- [Intermediate] [How do you decide the right sync frequency per connector when you have 60 connectors and a fixed cost budget?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11748_How_do_you_decide_the_right_sync_frequency_per_connector_whe.md>)
- [Intermediate] [Explain the trade-offs of running Airbyte syncs on a schedule vs event-driven triggers from Airflow sensors.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11811_Explain_the_trade-offs_of_running_Airbyte_syncs_on_a_schedul.md>)
- [Deep] [How would you decide the right sync frequency for a dashboard that needs near-real-time data?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11737_How_would_you_decide_the_right_sync_frequency_for_a_dashboar.md>)
- [Deep] [How do you version and promote connector configuration changes across dev, staging, and prod without manual clicking in the UI?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11759_How_do_you_version_and_promote_connector_configuration_chang.md>)
- [Deep] [Your incremental sync missed updates because the source app does in-place updates without touching updated_at. How do you make ingestion trustworthy?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11802_Your_incremental_sync_missed_updates_because_the_source_app_.md>)
- [Deep] [Explain how you'd migrate ingestion state so switching a connector from full-refresh to incremental doesn't re-sync everything.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11824_Explain_how_you_d_migrate_ingestion_state_so_switching_a_con.md>)

## Related concepts

- [Idempotent Incremental Loads](<../Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>) *(Idempotent Incremental Loads)*
- [Handling Source Changes and Spikes](<../ETL_ELT/Handling_Source_Changes_and_Spikes.md>) *(ETL ELT)*
- [Optimistic Concurrency and Clocks](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Reviewing AI Code for Performance Risks](<../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Performance_Risks.md>) *(AI Assisted Coding)*
- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*

## References

- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Datastore Connection Layer

