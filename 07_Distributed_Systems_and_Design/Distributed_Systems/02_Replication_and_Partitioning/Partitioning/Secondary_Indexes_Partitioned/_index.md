---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/02_Replication_and_Partitioning/Partitioning/Secondary_Indexes_Partitioned"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Secondary Indexes in Partitioned Data

**L5 Expert** · [↑ Partitioning Key-Value Data](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A partitioned database can index a non-key column in two ways: a local (document-partitioned) index that is cheap to write but needs scatter-gather reads, or a global (term-partitioned) index that reads from one partition but makes writes distributed and usually asynchronous.

### 1. The big picture
Key-value partitioning is simple because you route by primary key. Real applications also query by other attributes: "all red cars", "orders by customer", "articles mentioning *kafka*". Secondary indexes make this efficient on one node, but a secondary index does not map neatly to partitions: records matching one value of the indexed column live in many partitions. Two designs exist, and they are the same decision that Elasticsearch/Solr, Cassandra, DynamoDB, MongoDB and Kafka consumers all face.

### 2. Core concepts and vocabulary
- **Secondary index** — an index over a non-primary-key column; non-unique, returns many records.
- **Document-partitioned (local) index** — each partition maintains an index only over its own documents.
- **Term-partitioned (global) index** — one index covers all data, itself partitioned by *term* (the indexed value).
- **Scatter/gather** — send a query to all partitions and merge results.
- **Term** — a value being indexed (e.g., `color:red`, or a word in full-text search).
- **Asynchronous index update** — the global index is updated after the write commits, so it can be briefly stale.
- **Materialized view / denormalized lookup table** — manually maintained alternative to a global index.

### 3. How it works, step by step
**Local index (document-partitioned)** — used by MongoDB, Cassandra (secondary indexes), Elasticsearch, Solr, VoltDB, Riak search:
1. A write touches only the partition holding that document and updates that partition's local index.
2. A query by secondary attribute does not know which partitions hold matches, so it is sent to *all* partitions.
3. Each partition searches its local index; a coordinator merges results.
Writes are cheap and atomic; reads suffer tail-latency amplification (slowest partition governs) and scale poorly with partition count.

**Global index (term-partitioned)** — used by DynamoDB global secondary indexes, Riak search (older), Oracle global indexes:
1. The index is a separate structure partitioned by term (by range for range scans on terms, by hash for even load).
2. A write to a document may need to update index entries on several index partitions (one per indexed term), potentially on other nodes.
3. A query by term goes to the one index partition that owns it, which returns the matching primary keys; records are then fetched.
Reads are efficient (one partition plus fetches); writes are slower and more complicated, requiring a distributed transaction to be consistent, so global indexes are usually updated **asynchronously**.

### 4. Worked example
```text
Used cars, partitioned by car id across 3 nodes (P0, P1, P2).
Query: color = 'red'

Local indexes:     ask P0, P1, P2 -> each returns its red cars -> merge   (3 requests)
Global index by color:
   index partitions: [black..red] on N1,  [silver..white] on N2
   'red' lives only on N1 -> returns ids {191, 306, 768} -> fetch from P?   (1 index request + fetches)

Write car 893 (red, Audi):
   Local: update P1 only.
   Global: update P1 (record) and N1 (index 'red') and N2 (index 'Audi' if hashed elsewhere)
```
Reading it: local costs scale with the number of partitions at *read* time; global costs scale with the number of indexed attributes at *write* time. DynamoDB GSIs are eventually consistent for exactly this reason: a read may not see a write for a short time.

### 5. Under the hood
- **Why not a distributed transaction for the global index?** It would require 2PC across the record's partition and each index partition, reducing availability and write throughput; asynchronous maintenance accepts a staleness window instead (typically milliseconds to seconds, but unbounded under failures).
- **Local index detail in search engines**: each shard is a separate Lucene index; queries fan out to one replica per shard, then results are merged by score. Replicas help read throughput and tolerate slow shards via adaptive replica selection.
- **Term-partition choices**: partition by term range supports prefix and range queries on terms; by term hash gives even load. Full-text indexes combine many terms per document, so a multi-term query might hit several index partitions and intersect posting lists.
- **Index size**: global index posting lists for common terms are large and hot; rarely a single hot term overloads one partition.
- **Application-level alternative**: model a second table keyed by the attribute (e.g., `cars_by_color`) and write both, possibly with a transactional outbox or change stream for consistency.
- **Hybrid**: Cassandra recommends query-driven tables over secondary indexes; its newer SAI (storage-attached indexes) improve local index efficiency.

### 6. Performance and design implications
- Local-index query latency ≈ slowest of N partitions; add shards and tail latency rises, so cap shard counts and use replicas.
- Global index reads are fast and predictable, but write amplification (one write → several index writes) and index lag need monitoring.
- Choose index cardinality carefully: low-cardinality columns (boolean) produce huge posting lists; high-cardinality ones are expensive to maintain.
- Selective queries on partition key plus secondary attribute can route to one partition even with a local index; design the partition key to be part of the query where possible.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX ON t (col)` (Cassandra) | Local secondary index | Low-volume lookups within partition-key queries |
| DynamoDB GSI / LSI | Global (term-partitioned) / local index | Alternate access paths |
| Elasticsearch `number_of_shards`, `routing` | Shard count; route to a shard | Control scatter/gather fan-out |
| MongoDB `createIndex` on sharded collection | Local per-shard index | Queries including shard key avoid scatter |
| Materialized views / lookup tables | Hand-built global index | Predictable high-volume queries |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Local index: write-heavy workloads, queries usually include the partition key | Many partitions and queries without partition key |
| Global index: read-heavy secondary-attribute queries | Writes must be immediately consistent with index reads |
| Search engine for full-text and faceting | Strict transactional consistency required |
| Application-maintained lookup tables | You cannot manage dual writes safely |

### 9. Common mistakes
- Secondary index on a high-fan-out query without the partition key → scatter-gather across a large cluster → redesign the data model or key.
- Expecting global-index reads to be immediately consistent → missing rows → read from base table by key or accept staleness.
- Indexing low-cardinality flags → huge hot posting lists → use partial or composite indexes.
- Dual-writing lookup tables without idempotency → permanent drift → outbox or CDC.

### 10. 🎯 Interview kit
**30-second answer:** A local (document-partitioned) index is stored with each partition: writes are cheap but queries scatter to all partitions. A global (term-partitioned) index is partitioned by the indexed value: reads hit one partition but a write updates several partitions, so it's usually asynchronous and eventually consistent.

**Structure a longer answer:** why secondary indexes are hard → local approach → global approach → trade-offs (read vs write cost, consistency) → real systems and alternatives.

**Likely follow-ups:**
- *Why are DynamoDB GSIs eventually consistent?* → async maintenance avoids distributed transactions.
- *How would you reduce scatter-gather cost?* → include partition key in queries, fewer shards, replicas, or a global index.
- *What is a term?* → an indexed value or word.

**Signals interviewers look for:** read/write cost accounting and concrete system examples.

### 11. Check yourself
- [ ] Describe the lifecycle of a query under both index types.
- [ ] Explain the write path cost of a global index.
- [ ] Why do search engines use local indexes?
- [ ] Propose an application-level alternative to a global index.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Partitioning Secondary Indexes by Term
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Partitioning Secondary Indexes by Document
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Partitioning and Secondary Indexes
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Indexes, Partitioning, and Clustering
- *Database Internals* — Alex Petrov — Log-Structured Storage › Leveled compaction
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › From indexes to partitions and clustering
- *Designing Data-Intensive Applications* — Martin Kleppmann — I
- *Database Internals* — Alex Petrov — Log-Structured Storage › Size-tiered compaction
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — E
- *Designing Data-Intensive Applications* — Martin Kleppmann — Observing Derived State › Materialized views and caching

