---
id: "02_Data_Engineering/01_Lifecycle_and_Ingestion/Source_Systems"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Source Systems and Data Generation

**L4 Advanced** · [↑ Data Engineering Lifecycle and Ingestion](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Source systems — application databases, APIs, devices, files and queues — are where data is generated, and a data engineer must understand how each one creates, stores and exposes data before trying to ingest from it.

### 1. The big picture

Data engineering starts outside your control. Someone else built the application, the point-of-sale system, the IoT fleet or the third-party API; you consume what it emits, in the form and rhythm it chooses. Understanding *how* a source generates data — how it writes, commits, deletes, retries, versions its schema — determines whether ingestion is a quiet success or an endless fire drill.

Two ideas frame this lesson. First, operational systems (OLTP) and analytical systems are different animals: operational databases are tuned for many small, fast reads and writes and are closely guarded by their owners, so analytics usually goes to a *separate* system (a warehouse or lake) fed by extraction. Second, the most common interfaces you will meet are databases, APIs and files; each has quirks to learn. This lesson prepares you for the ingestion lessons that follow.

### 2. Core concepts and vocabulary

- **Source system** — originates data you ingest; usually owned by another team or vendor.
- **Analog vs digital data** — analog (speech, handwriting) is transient until digitised; digital data is native to or converted by a system.
- **Application database (OLTP)** — stores application state; many concurrent small reads/writes, low latency, high availability. "Transactional" does not always mean full ACID.
- **Data warehouse** — separate, read-optimised database holding a copy of data from many OLTP systems, in an analysis-friendly schema.
- **ETL** — extract from sources, transform, load into the warehouse.
- **DBMS** — storage engine + query optimiser + recovery and other management components.
- **API** — programmatic interface for exchanging data; most commonly HTTP-based.
- **REST** — stateless, resource-oriented HTTP API style using verbs like GET and PUT; each call is independent.
- **Client library / connector** — packaged code (often Python) that handles authentication and maps API calls to classes; off-the-shelf connectors save plumbing work.
- **Consistency model** — strong vs eventual; some stores offer per-request options.
- **Open source software (OSS)** — community-managed or commercial; evaluated on maturity, community, support, roadmap, and self-hosting cost.

### 3. How it works, step by step

How to approach any new source:

1. **Identify what it is**: relational DB, document DB, API, device fleet, queue, file drop.
2. **Read its documentation and learn its write path** — how it commits, updates, deletes, versions rows.
3. **Learn its access constraints**: rate limits, replicas, maintenance windows, whether extraction affects production performance.
4. **Map the data**: schema, keys, update semantics, timestamps, soft vs hard deletes.
5. **Choose an extraction method**: snapshot, incremental by timestamp, log-based CDC, API polling, file export, or event subscription.
6. **Agree a contract** with the owner: schema-change notice, retention, SLAs.
7. **Monitor** freshness, volume and quality from day one.

Database characteristics worth checking for any engine:

| Characteristic | Why it matters for ingestion |
|---|---|
| Lookups / indexes (B-tree, LSM) | Determine whether incremental queries by key or timestamp are cheap |
| Query optimiser | Predictability of extraction queries |
| Scaling (vertical vs horizontal) | Where to read from; replica availability |
| Modelling patterns | Normalised vs wide/nested shapes you'll receive |
| CRUD behaviour | How updates and deletes appear (in place, tombstones, versions) |
| Consistency | Whether a replica read can be stale |

### 4. Worked example

Source profile for an orders microservice backed by PostgreSQL, plus a payments REST API.

```text
SOURCE: orders-service (PostgreSQL 15, primary + 1 read replica)
  type            OLTP application database
  write rate      ~200 tx/s peak, ~30 tx/s average
  keys            order_id (bigint, monotonic), customer_id
  change marker   updated_at (set by app; NOT updated by manual fixes!)
  deletes         soft delete via is_deleted; hard deletes for GDPR requests
  extraction      logical replication slot -> CDC (no extra query load)
                  fallback: nightly snapshot from replica
  risk            long analytic queries on primary hurt checkout latency

SOURCE: payments-api (REST, JSON, OAuth2 client credentials)
  limit           100 req/min, pagination: cursor
  data            /v2/payments?updated_since=<ts>&cursor=<c>
  client library  vendor Python SDK (handles token refresh, retries)
  quirks          amounts in minor units (cents); timestamps in UTC but offset omitted
  late data       refunds can appear up to 30 days later
```

Reading it: each line is a decision input. Because `updated_at` is not reliable for manual fixes, CDC from the log is preferred; because the API has a rate limit and late refunds, the job must page with a cursor and re-read a 30-day trailing window.

### 5. Under the hood

**Why separate OLTP from analytics.** OLTP databases favour indexes and row storage that find and modify a few records quickly. Analytical queries scan huge ranges and aggregate; run against the production database they consume resources and degrade concurrent transactions, so owners protect these systems. The warehouse exists to give analysts a place to query freely, with a layout (often columnar) that suits scans. Small companies with few systems and small data may not need one — a conventional database or even a spreadsheet may suffice.

**Application database nuances.** A relational engine can read or update a row in about a millisecond (excluding network) and handle thousands of operations per second; document stores can commit even faster at the expense of potential inconsistency; some graph databases handle transactional use as well. "Transactional database" is shorthand for the OLTP workload shape and does not guarantee atomic multi-statement transactions in every product.

**REST in practice.** REST interactions are stateless: no server-side session or working directory; calls can change global state. REST is not a full specification, so each API requires domain learning. API quality varies from thin wrappers over internals to carefully engineered data endpoints. Even with libraries and managed connectors, teams usually maintain some custom API code: understanding the structure, writing extraction logic, handling pagination, retries, and deciding on a storage layout for responses.

**Choosing open source sources/tools.** Look at mindshare (stars, forks, commits), maturity, troubleshooting support, issue handling, sponsoring company, roadmap, and whether you can self-host (TCO and opportunity cost versus a managed offering).

### 6. Performance and design implications

- **Never analyse on the primary**: use replicas, CDC, or scheduled exports.
- **Index awareness**: incremental filters on non-indexed columns force full scans on the source.
- **Rate limits**: design for pagination, backoff and quotas; schedule heavy backfills outside business hours.
- **Schema drift**: new columns, type changes and renamed fields are normal; detect them automatically.
- **Time handling**: capture time zones, clock skew, and event vs update time.
- **Measure**: extraction duration, source CPU/IO during extraction, API quota usage, lag between source commit and landing, error and retry counts.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| Read replica connection | Offloads reads from the primary | Batch extraction |
| Logical replication / binlog / CDC connector | Streams committed changes | Low-impact incremental capture |
| `EXPLAIN` on extraction query | Shows index use and scan cost | Before scheduling a heavy extract |
| `curl -H "Authorization: Bearer …"` / SDK call | Tests an API endpoint and pagination | API discovery and debugging |
| Rate-limit headers (`Retry-After`, quota) | Tell you when to back off | Robust API clients |
| Vendor SDK / managed connector | Handles auth and pagination | Standard SaaS sources |
| Source-owner SLA / contract doc | Defines change notice and retention | Every production source |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Log-based CDC — need low latency and deletes; source supports it | Source is a managed DB without log access or the team can't operate the connector |
| Timestamp incremental — simple, `updated_at` reliable and indexed | Hard deletes and manual fixes bypass the timestamp |
| Full snapshot — small table, no change marker | Large tables or tight windows |
| REST API with SDK — standard SaaS | Rate limits are tight and volume is high; consider bulk export endpoints |
| File drop — partner can't provide API/DB access | Need freshness and schema enforcement |
| Self-hosted OSS — strong skills, special requirements | No capacity to patch, scale and monitor |

### 9. Common mistakes

- **Running heavy analytics on the production DB** → customer-facing slowdowns → replica or warehouse.
- **Trusting `updated_at` blindly** → missed changes → reconcile counts/hashes or use CDC.
- **Ignoring the source's write semantics** → partial or inconsistent reads → understand commits, isolation, replica lag.
- **No contract with the source owner** → surprise schema changes → agree notice periods and automate detection.
- **Hard-coding API quirks** (units, time zones) in many places → inconsistency → normalise once on landing.
- **Hand-building every connector** → maintenance burden → use SDKs and managed connectors where they exist.
- **Assuming APIs return all history** → silent gaps → check retention and paging limits.

### 10. 🎯 Interview kit

**30-second answer:** "A source system is wherever data is generated: application databases, APIs, devices, files, queues. Before ingesting I learn how it writes and changes data, its schema and delete behaviour, its limits, and the load extraction would add. OLTP sources are protected, so I extract via replicas, CDC or exports into a warehouse rather than querying production."

**Structure a longer answer:** clarify the source type and constraints → how it generates/changes data → extraction options (snapshot, incremental, CDC, API, file) → trade-offs → scenario.

**Likely follow-ups:**
- *Why not query the OLTP DB directly for analytics?* → Different workload; scans harm transactions; separate warehouse is optimised for analysis.
- *How do you extract from a REST API reliably?* → SDK/connector, pagination via cursor, rate-limit backoff, incremental `updated_since`, trailing window for late changes.
- *What does "stateless" mean in REST?* → Each request carries everything needed; no server session state.
- *How do you pick an OSS tool?* → Maturity, community, support, roadmap, TCO vs managed.

**Signals interviewers look for:** respect for source constraints, practical extraction choices, defensive handling of change and lateness.

### 11. Check yourself

- [ ] Explain why OLTP systems are separated from the analytical warehouse.
- [ ] List six properties of a database you would check before ingesting from it.
- [ ] Describe the REST principle of statelessness.
- [ ] Design an extraction plan for an API with rate limits and late-arriving corrections.
- [ ] Explain why `updated_at` can be an unreliable change marker.
- [ ] Name criteria for evaluating an open-source project before adopting it.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › APIs
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Source System Practical Details
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Data Warehousing
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Sources of Data: How Is Data Created?
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Open Source Software

