---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores"
tools: ["AWS"]
---

# DynamoDB Modeling and Capacity

**L4 Advanced** · [↑ Wide Column Stores](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** DynamoDB is a managed key-value/wide-column store where you design keys around known access patterns, pay in read/write capacity units, and must keep traffic spread across partitions because each partition has a hard throughput ceiling.

### 1. The big picture
DynamoDB hides servers, replication and compaction, so most of the engineering is in two places: **modeling** (choosing partition and sort keys, indexes and item shapes so every access pattern is a single `Query`) and **capacity** (understanding read/write units, throttling and hot partitions). It has no joins and no ad-hoc SQL planner; a query you did not design a key for becomes a `Scan`, which reads, and bills for, the whole table.

Compared with Cassandra (previous lesson), the data model looks similar (partition key + sort key), but operations differ: replication is fixed at three AZs, consistency is a per-request choice between eventual and strong reads, and the cost model is explicit.

### 2. Core concepts and vocabulary
- **Item** — one row, max 400 KB, a bag of attributes; only the key attributes are required.
- **Partition key (PK / hash key)** — hashed to choose the physical partition.
- **Sort key (SK / range key)** — orders items within one partition key value; enables range queries.
- **Item collection** — all items sharing a partition key value.
- **RCU / WCU** — read and write capacity units (below).
- **Provisioned vs on-demand mode** — you set RCU/WCU (optionally with auto scaling) or pay per request.
- **GSI (global secondary index)** — an index with its own partition/sort key and its own capacity; replicated asynchronously; eventually consistent reads only.
- **LSI (local secondary index)** — same partition key as the table, different sort key; shares the table's capacity; supports strongly consistent reads; must be defined at table creation; imposes a 10 GB limit per item collection.
- **Projection** — which attributes are copied into an index: `KEYS_ONLY`, `INCLUDE`, `ALL`.
- **Condition expression** — a predicate evaluated atomically on the item server-side before a write.
- **Burst / adaptive capacity** — mechanisms that absorb short spikes and uneven partitions.
- **LastEvaluatedKey** — pagination cursor returned by `Query`/`Scan`.

### 3. How it works, step by step
**Capacity math**
- 1 WCU = one write per second of an item up to **1 KB**. Larger items round up to the next KB.
- 1 RCU = one **strongly consistent** read per second of up to **4 KB**; an *eventually consistent* read costs half; a *transactional* read costs double.
- Writes: transactional writes cost double.

**Worked sizing.** 3,000 writes/s of 3 KB items: each write is 3 WCU → **9,000 WCU**. 8,000 strongly consistent reads/s of 12 KB items: each read is 12/4 = 3 RCU → **24,000 RCU**. If the reads could be eventually consistent, 12,000 RCU. Each GSI adds its own write cost on top of the base table (below).

**Request path.** A request goes to a request router, which hashes the partition key, finds the partition's leader replica (writes) or any replica (eventual reads), and checks the partition's share of table throughput. If the partition is over its limit and burst/adaptive capacity cannot cover it, you get `ProvisionedThroughputExceededException` (or a throttle in on-demand).

**Partition limits.** A single partition serves at most about **3,000 RCU and 1,000 WCU** per second regardless of how much you provision for the table, so one hot key cannot be fixed by buying capacity.

**Burst and adaptive capacity**
1. *Burst:* unused capacity from the last ~5 minutes can be consumed during short spikes.
2. *Adaptive:* DynamoDB can shift unused throughput toward partitions that are hotter than their share and isolate frequently accessed items onto their own partitions.
3. Both are best-effort smoothing, not a guarantee. A **persistent** hot key will still hit the 1,000 WCU / 3,000 RCU partition ceiling. Redesign the key.

### 4. Worked example
Access patterns for an order system: (1) get customer, (2) list customer's orders newest first, (3) get one order with its lines, (4) list orders by status.
```text
Table: PK = pk (string), SK = sk (string)
pk=CUST#42   sk=PROFILE                 name, email
pk=CUST#42   sk=ORDER#2026-10-12#o991   total, status
pk=ORD#o991  sk=HEADER                  customer_id, status, total
pk=ORD#o991  sk=LINE#001                sku, qty
GSI1: PK = status, SK = created_at      -- list by status
```
```python
# list latest 20 orders for a customer (pattern 2)
table.query(
    KeyConditionExpression=Key('pk').eq('CUST#42') & Key('sk').begins_with('ORDER#'),
    ScanIndexForward=False, Limit=20)

# pre-join: header and all lines in ONE round trip (pattern 3)
table.query(KeyConditionExpression=Key('pk').eq('ORD#o991'))

# optimistic concurrency with a version attribute
try:
    table.update_item(
        Key={'pk': 'ORD#o991', 'sk': 'HEADER'},
        UpdateExpression='SET #s = :new, version = version + :one',
        ConditionExpression='version = :expected',
        ExpressionAttributeNames={'#s': 'status'},
        ExpressionAttributeValues={':new': 'SHIPPED', ':one': 1, ':expected': v})
except ClientError as e:
    if e.response['Error']['Code'] == 'ConditionalCheckFailedException':
        ...  # re-read, re-apply the change, retry with backoff + jitter
```
The "pre-join" works because the header and its lines share a partition key and are stored contiguously, ordered by sort key, so one `Query` returns them together. Its limits: the item collection must stay bounded (keep it well under tens of MB; with an LSI it is hard-capped at 10 GB), results page at 1 MB, and entities that belong to several parents (many-to-many) need duplication or a GSI.

**Validating a single-table design before coding:** write each access pattern as a table: operation, key condition, filter, sort, expected frequency and size; then show for each which table or index key serves it. Any row that needs a filter on a non-key attribute or a `Scan` is a design gap.

### 5. Under the hood
- **Storage and replication.** Partitions are B-tree-backed, replicated across three AZs; writes go through a Paxos-based leader and are acked after a majority persists the write. Eventually consistent reads can hit any replica and may be briefly stale; strongly consistent reads go to the leader.
- **Partition splitting.** Partitions split by size (about 10 GB) or by throughput; this is why throughput per partition is not related to the table's total provisioning in a simple way.
- **GSI write path.** A write to the base table is propagated asynchronously to each GSI; if a GSI is under-provisioned it throttles *base table writes*. Each GSI that includes the changed attributes costs extra write units, so write cost is roughly (1 + number of affected indexes) × item size. Keep GSIs few, and project `KEYS_ONLY` or `INCLUDE` where possible to limit size, at the cost of a base-table fetch when you need other attributes.
- **FilterExpression.** Applied after the read. RCUs are charged for everything read from the key range, not for what survives the filter; latency can worsen because more data is read and discarded. Use key design or a GSI instead.
- **Pagination.** `Limit` caps items *evaluated* per page; with filters, a page can return fewer items than `Limit` even when more match, and `LastEvaluatedKey` is the only reliable "more data" signal. Loop until it is absent.
- **DAX.** An in-memory write-through cache cluster in front of DynamoDB for reads. Item cache and query cache are separate; the query cache is not invalidated by writes to items, only by TTL, so a `Query` result can be stale up to the cache TTL. It serves eventually consistent reads; strongly consistent reads pass through to DynamoDB.
- **Streams and exports.** Streams give an ordered per-item change log (24 h) for CDC; **export to S3** reads from point-in-time backups without consuming table capacity, and **import from S3** creates a new table (it cannot load into an existing one), with no table capacity consumed and a cost per GB.

### 6. Performance and design implications
- Spread writes: high-cardinality partition keys; for a hot logical key use **write sharding** (`counter#<0..N-1>`) and aggregate on read.
- Hourly `ProvisionedThroughputExceededException` spikes almost always mean a scheduled job or time-based key: check CloudWatch `ThrottledRequests`, then **CloudWatch Contributor Insights** to find the hot keys. Fix by smoothing the job, key salting or key re-design; raising capacity helps only if the per-partition ceiling is not the limiter.
- Large JSON blobs: you pay RCU for the whole item even if you need one field, and an update rewrites the item (WCU based on the whole item size). Prefer separate typed attributes for fields you filter, project or update; keep rarely used payloads in a separate item or S3.
- Ever-growing list attribute in one item: will hit the 400 KB limit and every append rewrites the item. Use one item per entry under the same partition key.
- Retry policy: exponential backoff with **full jitter** and a retry budget; the SDK already retries throttles, so stacking custom retries on top creates storms.
- Scan on a big table: use **parallel scan** (`Segment`/`TotalSegments`), small page sizes and rate limiting, or export to S3 and use Athena instead.
- Geospatial: store geohash prefix as partition key; choose precision so that a "nearby" query touches a small, fixed number of neighbouring cells.
- Money: use the Number type with integers in minor units (cents) or a string/decimal convention; do not use floating-point.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `Query` | Reads by PK (+ SK condition) | Default access path |
| `Scan` | Reads whole table/index | Admin/export only |
| `GetItem` / `BatchGetItem` | Key lookups | Single or multi-key fetch |
| `UpdateItem` + `ConditionExpression` | Atomic conditional write | Optimistic locking, idempotency |
| `TransactWriteItems` | All-or-nothing across items (up to 100) | Multi-item invariants; costs 2× |
| `ConsistentRead=True` | Strongly consistent read | Read-after-write on base table |
| Provisioned + auto scaling / On-demand | Capacity mode | Predictable vs spiky traffic |
| TTL attribute | Free background expiry | Sessions, ephemeral data |
| Export to S3 / Import from S3 | Bulk movement without table load | Analytics, new-table loads |
| PartiQL | SQL-like surface over the same API | Convenience; still check the access path |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Single-table design: few round trips, known access patterns | Access patterns still unknown or changing a lot |
| On-demand mode: spiky/unknown traffic | Steady high throughput, where provisioned is cheaper |
| GSI for another access path | Write-heavy tables where each GSI multiplies write cost |
| LSI for strongly consistent alternative sort order | Collections that can exceed 10 GB; LSIs cannot be added later |
| DAX for read-heavy, skew-tolerant workloads | Read-after-write correctness or `Query` freshness needs |
| `TransactWriteItems` for invariants | Hot items: contention and 2× cost |
| Global tables for multi-region active-active | Cases that cannot accept last-writer-wins conflict resolution |

### 9. Common mistakes
- Timestamp or sequence as the partition key → all writes hit one partition → add a high-cardinality prefix or shard.
- Using `FilterExpression` to "narrow" a query → same RCUs, more latency → model the filter into a key or GSI.
- Assuming `Limit=10` returns 10 → fewer items with filters → page until `LastEvaluatedKey` is absent.
- Many GSIs "just in case" → write amplification and throttled base writes → consolidate and use overloaded GSI keys.
- Relying on burst/adaptive capacity for a hot key → throttles at the 1,000 WCU ceiling → re-key or shard.
- Eventually consistent read right after a write → missed item → use `ConsistentRead` on the table (not on a GSI).
- Unbounded item growth → 400 KB error → item-per-event model.
- Treating DAX as strongly consistent → stale reads → keep correctness-critical paths off DAX.

### 10. 🎯 Interview kit
**30-second answer:** DynamoDB is access-pattern-first: choose partition and sort keys so each query is one `Query`, add GSIs for other paths, keep key cardinality high because a partition tops out near 1,000 WCU / 3,000 RCU, and size capacity as 1 KB write units and 4 KB strongly consistent read units.

**Structure a longer answer:** list access patterns with volume → map each to table/GSI key → check item size and collection growth → compute capacity → identify hot-key risks and mitigations → consistency and concurrency (conditions, transactions) → operations (TTL, streams, export, backups).

**Likely follow-ups:**
- *WCU/RCU for 3,000 × 3 KB writes and 8,000 × 12 KB strong reads?* → 9,000 WCU, 24,000 RCU.
- *LSI vs GSI?* → LSI: same PK, shares capacity, strong reads, creation-time only, 10 GB collection cap. GSI: any keys, own capacity, eventual reads, async.
- *How do you do optimistic locking?* → version attribute + condition expression; catch `ConditionalCheckFailedException`, re-read, retry with jitter.
- *How do you run reporting?* → Streams to a warehouse or export to S3 + Athena, never `Scan` production.

**Signals interviewers look for:** key design from access patterns, correct unit math, knowing filters do not save capacity, hot-partition reasoning, and honest limits of DAX and global tables.

### 11. Check yourself
- [ ] Do the RCU/WCU math for a table with 6 KB items, 500 writes/s and 2,000 eventually consistent reads/s.
- [ ] Design keys for chat messages (ordered, paginated, bounded partitions).
- [ ] Explain why a FilterExpression does not reduce consumed capacity.
- [ ] Explain the difference between a short spike and a persistent hot partition and what DynamoDB does for each.
- [ ] Implement a version-based conditional update with retry.
<!-- explainer:end -->

## Go deeper

- [Cassandra Replication and Read Path](<Cassandra_Replication_and_Read_Path.md>)
- [Wide Column Design and Migration](<Wide_Column_Design_and_Migration.md>)

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Describe how DynamoDB import/export from S3 works and when you'd use it to bulk-load a new table vs writing through the API. What are the ordering, cost, and downtime implications?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05800_Describe_how_DynamoDB_import_export_from_S3_works_and_when_y.md>)
- [Deep] [How do you validate that a proposed DynamoDB single-table design actually satisfies ALL known access patterns before writing code? Walk me through your access-pattern-to-key mapping process.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05784_How_do_you_validate_that_a_proposed_DynamoDB_single-table_de.md>)
- [Deep] [Explain how you'd structure keys so that a single Query can fetch a parent entity and all its children in one round-trip (the 'pre-join' pattern) in DynamoDB. What are the limits of this pattern?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05803_Explain_how_you_d_structure_keys_so_that_a_single_Query_can_.md>)
- [Deep] [Design a single-table DynamoDB schema for an e-commerce order system that must serve: get order by id, list orders by customer sorted by date, and list orders by status. Show the PK/SK and any GSIs.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05925_Design_a_single-table_DynamoDB_schema_for_an_e-commerce_orde.md>)
- [Deep] [What's the difference between DynamoDB's Query and Scan cost model, and why does adding a FilterExpression not reduce consumed capacity? How does this change your data model?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05951_What_s_the_difference_between_DynamoDB_s_Query_and_Scan_cost.md>)
- [Deep] [How do you handle a schema-less write where different item types share a table in DynamoDB (single-table design)? How do you keep application code sane and avoid attribute-name collisions?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05984_How_do_you_handle_a_schema-less_write_where_different_item_t.md>)
- [Deep] [Your org's DynamoDB tables have accumulated 200+ GSIs across services with nobody tracking which are used. Design a program to audit, deprecate, and consolidate them safely, and quantify the write-amplification savings to justify the effort to leadership.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05857_Your_org_s_DynamoDB_tables_have_accumulated_200_GSIs_across_.md>)
- [Expert] [Design an idempotent payment-write path on DynamoDB so a client retry never double-charges. What key and condition expression do you use, and how does TTL interact with your idempotency records?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05890_Design_an_idempotent_payment-write_path_on_DynamoDB_so_a_cli.md>)

## Related concepts

- [Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*
- [Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [Optimistic Concurrency and Clocks](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*
- [Large Values and TOAST](<../../../SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [Choosing a Database for the Workload](<../../../Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Random Keys and Index Locality](<../../../Query_Performance/Indexing/Random_Keys_and_Index_Locality.md>) *(Indexing)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Kinesis Data Streams Versus Amazon MSK › Streaming Amazon DynamoDB data into a centralized data lake
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Expiring Snapshots from Open Table Formats › Archiving Data from Amazon DynamoDB to Amazon S3
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Modeling Strategies for Amazon DynamoDB
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Modeling Strategies for Amazon DynamoDB › Core concepts of DynamoDB
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Building Native Applications with Delta Lake › DynamoDB lock
- *Designing Data-Intensive Applications* — Martin Kleppmann — Detecting Concurrent Writes
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Consistency in Amazon DynamoDB
- *Ultimate AWS Data Engineering* — Rathish Mohan — Exploring the Realm of AWS Data Storage Services › Amazon DynamoDB: NoSQL for Speed and Scalability
- *Ultimate AWS Data Engineering* — Rathish Mohan — Exploring the Realm of AWS Data Storage Services › Unveiling DynamoDB's Key Features: Fast Performance and High Availability
- *Ultimate AWS Data Engineering* — Rathish Mohan — Exploring the Realm of AWS Data Storage Services › Designing Data Models in DynamoDB
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Additional Resources
- *Building Data Integration Solutions* — Jay Borthen — Data Stores and Management Systems › Amazon DynamoDB and DocumentDB

