---
id: "03_Big_Data_and_Streaming/01_Batch_Processing/Batch_Output_and_Derived_Data"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Batch Output and Derived Data

**L4 Advanced** · [↑ Batch Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The output of a batch job is usually a *derived dataset* (search index, key-value database, model, warehouse table) that is built as immutable files and then bulk-loaded into a serving system — never written record by record from inside the job.

### 1. The big picture

Batch jobs are not just "reports". A workflow of MapReduce or Spark jobs is closer to analytics in that it scans large inputs, but its output is typically another *structure*: a search index, a database of recommendations keyed by user ID, a trained classifier, a cleaned and modelled table loaded into a warehouse.

How that output gets from the batch system (Hadoop, Spark, a lake) to the application that serves users is where careless designs go wrong. The good pattern keeps the batch philosophy intact — immutable inputs, no side effects, outputs replaced wholesale — which gives you cheap reruns, rollbacks and experimentation. This lesson covers that pattern, why writing straight to a production database from tasks is a bad idea, and how batch systems relate to MPP databases and the "one big derived-data database" view of an organisation.

### 2. Core concepts and vocabulary

- **Derived data** — datasets computed from other data (indexes, aggregates, features, caches); can always be rebuilt from the source of truth.
- **Bulk load** — building database files offline and installing them in a read-only serving cluster in one step.
- **Human fault tolerance** — the ability to recover from buggy code by rolling code back and rerunning, because outputs are regenerable.
- **Minimising irreversibility** — design so mistakes are cheap to undo; encourages faster iteration.
- **MPP database** — massively parallel processing database (Teradata, Gamma, Tandem NonStop SQL): parallel execution of analytic SQL on a cluster with proprietary storage and up-front modelling.
- **Schema-on-read / "sushi principle"** — dump raw data first and interpret it later; "raw data is better".
- **Data lake / enterprise data hub** — central store of raw data in its original form.
- **Federated database (polystore)** — a unified query interface over several storage engines (e.g., PostgreSQL foreign data wrappers); unifies reads.
- **Unbundled database** — composing specialised systems and keeping them in sync with change data capture and event logs; unifies writes.
- **Sqoop** — Hadoop tool that moves data between relational databases and HDFS in both directions.

### 3. How it works, step by step

**Safe publication of batch output (key-value store example)**

1. The batch job computes results (e.g., "related products by product ID").
2. Inside the job, it builds the database's own file format (partitioned, indexed files) and writes them to an output directory in the distributed filesystem.
3. The job finishes; its output is all-or-nothing — successful tasks' results are committed once, failed attempts are discarded.
4. Serving nodes copy these immutable files and load them as a new version; reads switch from the old version to the new in one step.
5. The previous version is kept so a bad build can be rolled back by switching back.

**Unsafe alternative:** each mapper/reducer calls the production database client for each record. Why it breaks:

| Problem | Consequence |
|---|---|
| One network round trip per record | Orders of magnitude slower than batch throughput, even with client batching |
| Many parallel tasks hammer one database | Overload, degraded user-facing queries, outages elsewhere |
| Side effects visible before job completes | Partial/duplicated results from failed attempts and speculative execution |

### 4. Worked example

A nightly recommendation pipeline:

```text
03:00  Spark job reads yesterday's events (immutable, date-partitioned) from the lake
03:30  Computes (product_id -> top 20 related products)
04:00  Writes /derived/related_products/dt=2026-10-13/ in the store's native file format
04:10  Quality job compares row count and distribution with dt=2026-10-12; fails the run on >5% drift
04:15  Serving cluster bulk-loads the directory and atomically swaps to the new version
```

```bash
# Roll back by switching the pointer to the previous directory - no data repair needed
ln -sfn /derived/related_products/dt=2026-10-12 /serving/related_products/current
```

Contrast with a buggy transactional update job: rolling back the code would not fix rows it already corrupted in a read-write database. In the batch design, you fix the code, rerun, and replace the output — or just keep serving the old version.

Moving relational data in and out of Hadoop (Sqoop):

```bash
sqoop import --connect jdbc:mysql://db/shop --table orders \
      --target-dir /raw/shop/orders/dt=2026-10-13
sqoop export --connect jdbc:mysql://db/shop --table daily_stats \
      --export-dir /derived/daily_stats/dt=2026-10-13
```

### 5. Under the hood

**Why this philosophy works.** It extends Unix: input untouched, previous output completely replaced, no other side effects. Consequences listed by the book:

- *Rollback and rerun* fix bugs; old output can sit in another directory and be switched back.
- *Faster development* because mistakes are not irreversible.
- *Automatic retry* is safe because inputs are immutable and failed task outputs are discarded.
- *Reuse*: the same files feed other jobs, including monitoring jobs that compare outputs with the previous run.
- *Separation of logic and wiring*: one team writes the job, another decides where and when to run it.

Unlike Unix, Hadoop avoids untyped text parsing by using structured formats — Avro and Parquet — with schema-based encoding and schema evolution.

**Hadoop versus MPP databases.** The parallel join and aggregation algorithms existed in MPP databases a decade before MapReduce. Differences: MPP systems execute analytic SQL against their own storage; HDFS + MapReduce is closer to a general-purpose operating system that can run arbitrary code (ML, image processing, text indexing) on any bytes. Files can hold any model or encoding — text, images, sensor data, feature vectors, genomes — so data can be dumped first and modelled later, while MPP systems demand up-front modelling and import. The cost of "dump now": consumers take on interpretation (schema-on-read). The benefit: faster centralisation and several views from the same raw data. Hadoop is therefore often the ETL stage: raw dump → MapReduce cleaning and modelling → load into an MPP/warehouse system.

**The meta-database of everything.** Seen at organisation scale, batch, stream and ETL processes act like index and materialised-view maintenance for one giant database; derived systems (search index, cache, warehouse, key-value store) are like different index types. Two ways to compose them: *federated* (read side — one query interface) and *unbundled* (write side — change data capture and event logs keep all systems in sync).

### 6. Performance and design implications

- **Build outputs offline, load in bulk**; load throughput then depends on file copy speed, not per-record writes.
- **Version outputs by date/run ID** and keep several previous versions to allow instant rollback.
- **Validate before publishing** (counts, null rates, distribution drift, sample lookups) and gate promotion on checks.
- **Isolate the serving store** from batch concurrency; control load rate on cutover.
- **Keep raw inputs** (the "sushi principle") so any derived dataset can be rebuilt with new logic.
- **Choose encodings** (Avro for row interchange, Parquet for analytics) that carry schemas and evolve safely.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `sqoop import / export` | Moves tables between RDBMS and HDFS | Initial loads, publishing results back |
| `hdfs dfs -mv` / `distcp` | Atomically renames / copies directories | Promoting a completed output directory |
| `_SUCCESS` marker file | Signals a job finished and output is complete | Consumers wait for it before reading |
| Output committer (`FileOutputCommitter`) | Commits task output on success only | Safe, all-or-nothing writes |
| Bulk-load tools (e.g., HBase `completebulkload`, store-specific loaders) | Install pre-built files into a serving store | Large daily/hourly refreshes |
| Date/run-ID partitioned paths | Version outputs | Rollback and auditing |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Build files in the job and bulk-load | Results must be visible within seconds of events (streaming/update-in-place instead) |
| Raw data dump + schema-on-read | Many consumers misinterpret data; add contracts, catalog and quality checks |
| MPP/warehouse for repeated SQL analytics | You need arbitrary computation (ML, text, images) |
| Hadoop/lake for ETL, then warehouse for serving | Duplicating data without ownership or lineage |
| Federated queries for exploration | Heavy joins across systems at high frequency |

### 9. Common mistakes

- Writing from tasks into the live database → overload, partial visibility, duplicate writes on retry → build files and bulk-load.
- Overwriting the previous output in place → no rollback → write to new versioned path, then swap.
- Publishing without checks → bad data reaches users → add automated validation against the last run.
- Modelling everything up front before landing data → slow integration → land raw, model downstream.
- Forgetting ownership of raw dumps → "data swamp" → catalog, schema and retention policies.

### 10. 🎯 Interview kit

**30-second answer:** Batch output is typically derived data — indexes, key-value stores, models. The right way to publish it is to build immutable files inside the job and bulk-load them into the serving system, because per-record writes are slow, overload the database, and leak partial results from retries. Immutable inputs and replaced outputs also give cheap rollback and rerun.

**Structure a longer answer:** what batch produces → why not write directly → bulk-load/version/swap → human fault tolerance → compare with MPP and warehouses → raw-first design.

**Likely follow-ups:**
- *How do you roll back a bad batch release?* → Switch to the previous output version or fix code and rerun.
- *Hadoop vs MPP database?* → MPP: SQL on proprietary storage with up-front modelling; Hadoop: general programs over files, schema-on-read.
- *What is the sushi principle?* → Raw data is better; transform later for multiple purposes.
- *How do you keep several derived systems consistent?* → Change data capture / event logs (unbundled database).

**Signals interviewers look for:** you reason about side effects and idempotence, value reproducibility and rollback, and understand the lake-then-warehouse flow.

### 11. Check yourself

- [ ] List three reasons not to write to a production DB from mappers or reducers.
- [ ] Describe a versioned publish-and-swap procedure and its rollback.
- [ ] Explain "human fault tolerance" and how a read-write database lacks it.
- [ ] Contrast schema-on-read with up-front modelling and name a downside of each.
- [ ] Explain federated vs unbundled databases.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — G › H
- *Designing Data-Intensive Applications* — Martin Kleppmann — Map-Side Joins › Philosophy of batch process outputs
- *Designing Data-Intensive Applications* — Martin Kleppmann — Comparing Hadoop to Distributed Databases › Diversity of storage
- *Designing Data-Intensive Applications* — Martin Kleppmann — B
- *Designing Data-Intensive Applications* — Martin Kleppmann — Unbundling Databases › The meta-database of everything
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › The Dataflow Model and Unified Batch and Streaming
- *Designing Data-Intensive Applications* — Martin Kleppmann — Comparing Hadoop to Distributed Databases
- *Designing Data-Intensive Applications* — Martin Kleppmann — Map-Side Joins › Key-value stores as batch process output
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — H
- *Hadoop: The Definitive Guide* — Tom White — Sqoop › Chapter 15. Sqoop

