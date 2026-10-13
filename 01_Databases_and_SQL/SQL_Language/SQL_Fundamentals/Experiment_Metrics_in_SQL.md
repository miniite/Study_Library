---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Experiment_Metrics_in_SQL"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Experiment Metrics in SQL

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Storing and querying experiment, evaluation and telemetry results in SQL comes down to a few design habits — one fact row per measurement at a clear grain, typed columns for what you filter on, JSON only for the long tail, and aggregates (mean, percentiles, confidence interval) computed with SQL so a real change can be told from noise.

### 1. The big picture
Even apps built around an LLM API and a vector store need relational data: prompts and versions, requests and responses, eval scores, latencies, costs, feature flags and experiment arms. SQL is where you answer "did quality drop after prompt v7 shipped?" or "which model has the worst time-to-first-token under load?". The questions are generic *experiment-metrics* questions: assign units to arms, record measurements, aggregate per arm, compare.

The design work is mostly **schema**: choosing the grain, the keys, how to store enumerations and semi-structured payloads, how to partition fast-growing tables, and how to keep read queries (unread badges, latest-version lookups) cheap.

### 2. Core concepts and vocabulary
- **Grain** — what one row represents (one request; one judge score per request per metric).
- **Fact table** — measurements at a grain; **dimension** — descriptive entities (model, prompt version).
- **Arm / variant** — an experiment group (control, treatment).
- **Metric** — a computed measurement (mean score, p95 latency, pass rate).
- **Confidence interval (CI)** — range for the true mean; approx. `mean ± 1.96·sd/√n`.
- **TTFT** — time to first token; **inter-token latency** — gap between streamed tokens.
- **Enum storage** — `smallint` code, text, or lookup table with FK.
- **JSON column** — semi-structured payload stored as text/`json`/`jsonb`.
- **Promotion** — moving a frequently-filtered JSON field into a real, indexed column.
- **Version set** — the set of tool/prompt versions active for a deployment at a time.

### 3. How it works, step by step
**Schema for LLM-judge evaluations:**
1. `eval_run(run_id, model_version, prompt_version_id, started_at, dataset_id)`.
2. `eval_result(run_id, example_id, metric, score, reason, judged_at)` with PK `(run_id, example_id, metric)`.
3. Index for the common read: `(prompt_version_id, started_at)` on runs; `(metric, run_id)` on results.

**Comparing two versions:**
1. Aggregate per version: `n`, `AVG(score)`, `STDEV(score)`.
2. Compute CI half-width `1.96 * sd / SQRT(n)`.
3. If CIs overlap substantially, treat as noise; otherwise a candidate regression (use a proper test downstream for decisions).

**Attributing a drop to a prompt change:** join answers to the prompt-version table on `prompt_version_id` and compare score distributions *before and after* the deploy timestamp, or better, by version directly.

### 4. Worked example
```sql
-- Schema
CREATE TABLE dbo.prompt_version (
    prompt_version_id int IDENTITY PRIMARY KEY,
    prompt_name       varchar(100) NOT NULL,
    version_no        int          NOT NULL,
    deployed_at       datetime2    NOT NULL,
    retired_at        datetime2    NULL,
    CONSTRAINT UQ_prompt UNIQUE (prompt_name, version_no)
);
CREATE TABLE dbo.llm_request (
    request_id        bigint IDENTITY PRIMARY KEY,
    ts                datetime2 NOT NULL,
    model             varchar(50) NOT NULL,
    prompt_version_id int NOT NULL REFERENCES dbo.prompt_version,
    ttft_ms           int NULL,
    total_ms          int NULL,
    tokens_in         int NULL, tokens_out int NULL,
    payload           nvarchar(max) NULL,     -- raw request JSON
    CONSTRAINT CK_payload_json CHECK (payload IS NULL OR ISJSON(payload) = 1)
);
CREATE INDEX IX_req_ver_ts ON dbo.llm_request (prompt_version_id, ts) INCLUDE (ttft_ms);

-- Per-version quality with CI (judge scores 0..1)
SELECT pv.prompt_name, pv.version_no,
       COUNT(*)                                   AS n,
       AVG(r.score)                               AS mean_score,
       1.96 * STDEV(r.score) / SQRT(COUNT(*))     AS ci95_halfwidth
FROM   dbo.eval_result r
JOIN   dbo.llm_request q  ON q.request_id = r.request_id
JOIN   dbo.prompt_version pv ON pv.prompt_version_id = q.prompt_version_id
WHERE  r.metric = 'helpfulness'
GROUP  BY pv.prompt_name, pv.version_no
ORDER  BY pv.prompt_name, pv.version_no;

-- Worst models by p95 TTFT during the busiest hour
WITH busy AS (
    SELECT TOP (1) DATEADD(HOUR, DATEDIFF(HOUR,0,ts), 0) AS h
    FROM dbo.llm_request GROUP BY DATEADD(HOUR, DATEDIFF(HOUR,0,ts), 0)
    ORDER BY COUNT(*) DESC
)
SELECT DISTINCT model,
       PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY ttft_ms) OVER (PARTITION BY model) AS p95_ttft
FROM   dbo.llm_request q CROSS JOIN busy b
WHERE  q.ts >= b.h AND q.ts < DATEADD(HOUR,1,b.h) AND q.ttft_ms IS NOT NULL
ORDER  BY p95_ttft DESC;

-- Read JSON field, then promote it when queries need it
SELECT request_id, JSON_VALUE(payload, '$.temperature') AS temperature
FROM   dbo.llm_request WHERE JSON_VALUE(payload,'$.user_tier') = 'pro';
ALTER TABLE dbo.llm_request ADD user_tier AS CAST(JSON_VALUE(payload,'$.user_tier') AS varchar(20)) PERSISTED;
CREATE INDEX IX_req_tier ON dbo.llm_request (user_tier, ts);
```
How to read the CI query: if version 6 has mean 0.81 ±0.01 and version 7 has 0.78 ±0.01, the intervals don't overlap, so the drop is unlikely to be noise; with ±0.04 it could be.

### 5. Under the hood (optional at L4)
`JSON_VALUE` on a text column parses JSON per row and can't use an index; a **persisted computed column** materializes the extracted value so it can be indexed and the plan can seek — this is the standard promotion path in SQL Server (PostgreSQL's `jsonb` supports GIN indexes and expression indexes instead). Enum storage trade-off: `smallint` codes are 2 bytes and cheap to join but opaque; text is readable but wider and typo-prone; a lookup table with a foreign key gives both readability (via join) and enforcement, with a small join cost that is almost always irrelevant against I/O. `PERCENTILE_CONT` is a window-only function in T-SQL (needs `OVER`) and usually sorts per partition — pre-aggregating or sampling helps on huge tables.

### 6. Performance and design implications
- **Notifications at billions of rows:** partition by time, store `is_read`, keep a small per-user counter table (`unread_count`) updated transactionally or a filtered index `WHERE is_read = 0`; archive old partitions.
- Choose the grain deliberately; adding a metric as a *row* (`metric` column) scales better than a new column per metric.
- Index on `(prompt_version_id, ts)` supports per-version time-range reads.
- Keep raw payloads out of hot tables (separate table or compressed column) so scans read only narrow rows.
- Precompute rollups (hour × model) for dashboards; compute CIs on the rollup using sum, sum of squares and count.
- Use `CHECK (ISJSON(...)=1)` to keep payloads parseable.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ISJSON`, `JSON_VALUE`, `JSON_QUERY`, `OPENJSON` | Validate / extract / shred JSON | Semi-structured payloads |
| Persisted computed column + index | Index a JSON field | Promotion |
| `PERCENTILE_CONT … OVER` | Percentiles | Latency SLOs |
| `STDEV`, `AVG`, `COUNT` | CI inputs | Version comparison |
| Filtered index `WHERE is_read=0` | Index the small hot subset | Unread badges |
| `DATA_COMPRESSION = PAGE` | Shrink wide tables | Payload-heavy logs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Typed columns | Attributes vary per record and are rarely filtered |
| JSON column | You filter/aggregate on a field constantly — promote it |
| Lookup table for enums | The set is tiny and truly fixed — `CHECK` constraint suffices |
| One row per (example, metric) | Fixed small metric set queried together — wide row may be simpler |

### 9. Common mistakes
- Averaging per-request means across arms of unequal size → wrong weighting → aggregate from raw rows or weight by `n`.
- Comparing means without `n` or CI → noise mistaken for regression → show both.
- Storing percentiles and averaging them → meaningless → store histograms or raw data.
- JSON everywhere → unindexable filters → promote hot fields.
- Using local time → deploy-time comparison skewed → store UTC.
- Unbounded log table without partitioning/retention → slow deletes → partition by time and `SWITCH`/drop.
- Attributing to a deploy time while versions overlap in rollout → use the per-request `prompt_version_id`.

### 10. 🎯 Interview kit
**30-second answer:** "I model one fact row per request or per judge score with typed, indexed columns for what I filter on and JSON for the rest, keep a version dimension, and compare versions with per-group n, mean and confidence interval computed in SQL."

**Structure a longer answer:** clarify (grain, volume, who reads it) → schema → queries (per-version CI, percentiles) → trade-offs (JSON vs columns, enum storage, partitioning) → scenario (a quality drop traced to prompt v7 by version join).

**Likely follow-ups:**
- *When promote a JSON field?* → once filtered/grouped regularly; use computed column + index.
- *smallint vs text vs lookup?* → size/readability/enforcement trade.
- *Cheap unread count at scale?* → counter table or filtered index; partition and archive.
- *Why not store p95 and average it?* → percentiles aren't additive.

**Signals interviewers look for:** grain thinking, honest handling of noise, indexing JSON properly.

### 11. Check yourself
- [ ] Design tables for LLM-as-judge scores with a primary key at the right grain.
- [ ] Write per-version mean ± CI and interpret overlap.
- [ ] Promote a JSON field to an indexed column.
- [ ] Explain why percentiles can't be averaged.
- [ ] Propose a cheap unread-badge design at billions of rows.
<!-- explainer:end -->

## Practice questions

- [Basic] [Why does a GenAI engineer still need SQL when the app mostly calls an LLM API and a vector database?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40721_Why_does_a_GenAI_engineer_still_need_SQL_when_the_app_mostly.md>)
- [Intermediate] [How would you store few-shot examples in SQL so a prompt-builder can fetch the k most relevant examples for a given input?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40756_How_would_you_store_few-shot_examples_in_SQL_so_a_prompt-bui.md>)
- [Intermediate] [How would you store the source system prompt version with each response so you can compare answer quality before and after a prompt change?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40794_How_would_you_store_the_source_system_prompt_version_with_ea.md>)
- [Intermediate] [Explain how you'd design a SQL schema for storing tool definitions (name, JSON schema, version) that an agent loads, and query the active version set for a given deployment.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40920_Explain_how_you_d_design_a_SQL_schema_for_storing_tool_defin.md>)
- [Deep] [How would you store streaming LLM output in SQL, and would you write each token or the final assembled response? Explain the trade-off.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40801_How_would_you_store_streaming_LLM_output_in_SQL_and_would_yo.md>)
- [Deep] [How would you write SQL to build a per-request feature-logging table so you can later reconstruct exactly what the model saw at inference time for debugging and replay?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40698_How_would_you_write_SQL_to_build_a_per-request_feature-loggi.md>)
- [Deep] [Explain how you'd store token-level streaming latency (time-to-first-token, inter-token latency) per request in SQL and query the models with the worst TTFT under load.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40855_Explain_how_you_d_store_token-level_streaming_latency_time-t.md>)
- [Deep] [How would you design a SQL query that joins a prompt-versions table to an answers-log table to attribute a quality drop to a specific prompt change and its deploy time?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40907_How_would_you_design_a_SQL_query_that_joins_a_prompt-version.md>)

## Related concepts

- [Activity Timeline Schema](<../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>) *(MySQL Platform)*
- [Modeling Relationships and Schemas](<../../Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*
- [Choosing a Database for the Workload](<../../Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*

## References

- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — SQL Graph Table Basics › Object Creation
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Tree Structures, Algorithms, and Performance › Performance Comparison
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Writing Clean, Maintainable, and Well-Documented T-SQL Code
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Option 1: Query Store (Recommended)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Enabling Query Store
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Query Store Components
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › SQL Comments
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Backup and Recovery › Object stores
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — A Data Architecture Sampler › CQRS
- *Building Medallion Architectures* — Piethein Strengholt — Build the Silver Layer › Implementation of the Metadata Store
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Automating Data Quality Monitoring with Machine Learning
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Managing Data with Delta Lake › Querying by version number

