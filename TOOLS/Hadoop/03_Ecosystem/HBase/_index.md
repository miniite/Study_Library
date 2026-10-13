---
id: "TOOLS/Hadoop/03_Ecosystem/HBase"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# HBase

**L4 Advanced** · [↑ Hadoop Ecosystem](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** HBase is a distributed, sorted, wide-column store on top of HDFS that gives you fast random reads and writes by row key over billions of rows — the thing HDFS and Hive on their own cannot do.

### 1. The big picture
HDFS is built for big sequential scans of immutable files; it cannot update row 42 in place or fetch one record in milliseconds. Many applications (user profiles, messaging, time-series, web-page stores) need exactly that over datasets too large for one machine. HBase, modelled on Google's Bigtable, fills the gap: it stores tables of rows sorted by key, splits them into **regions** spread over servers, and persists everything on HDFS.

The project began at Powerset in late 2006 after the Bigtable paper, shipped inside Hadoop 0.15 in 2007 and became an Apache top-level project in 2010. It reuses Hadoop's configuration system and filesystem API, so Hadoop knowledge transfers. Its internals (regions, WAL, MemStore, HFile) are covered in the architecture child lesson; ZooKeeper (a sibling lesson) coordinates it.

### 2. Core concepts and vocabulary
- **Table** — a collection of rows, sorted lexicographically by **row key** (a byte array).
- **Row key** — the *only* index; every read is "by key" or "by key range".
- **Column family** — a group of columns declared at table-creation time; each family is stored together on disk and tuned (compression, TTL, versions) separately.
- **Column qualifier** — the column name inside a family (created on the fly, no schema change); full column = `family:qualifier`.
- **Cell** — the value at (row, family, qualifier, timestamp).
- **Version / timestamp** — each cell can keep several timestamped versions; a read returns the newest by default.
- **Region** — a contiguous range of rows of one table; the unit of distribution and load balancing.
- **RegionServer** — a process that serves a set of regions.
- **HMaster** — assigns regions, handles schema changes and balancing; not on the read/write path.
- **`hbase:meta`** — catalogue table that maps regions to servers.

### 3. How it works, step by step
The conceptual model is a **multidimensional sorted map**:
`(row key, column family:qualifier, timestamp) → value`.

| Row key | `contents:html` | `anchor:cnnsi.com` | `anchor:my.look.ca` |
|---|---|---|---|
| `com.cnn.www` | `<html>…` (t3, t5, t6) | `"CNN"` (t9) | `"CNN.com"` (t8) |

Webtable idea from the Bigtable paper: reversed URLs as keys put pages from one domain next to each other.

How a client reads a row:
1. Ask ZooKeeper where `hbase:meta` is.
2. Look up in `hbase:meta` which region covers the key (largest start key ≤ row key) and which RegionServer hosts it; cache the answer.
3. Talk directly to that RegionServer for `Get`/`Put`/`Scan`. The master is not involved.

**Wide-column ≠ column-oriented.** In HBase each column family stores whole rows of that family together, with no column-wise compression — it is mostly row-oriented. Only the *families* are separated.

### 4. Worked example
```text
$ hbase shell
hbase> create 'users', 'info', 'activity'
hbase> put 'users', 'u0042', 'info:name', 'Asha'
hbase> put 'users', 'u0042', 'info:city', 'Pune'
hbase> put 'users', 'u0042', 'activity:last_login', '2024-05-01'
hbase> get 'users', 'u0042'
hbase> scan 'users', {STARTROW => 'u0040', STOPROW => 'u0050'}
hbase> disable 'users'
hbase> drop 'users'
```
What happens: `create` registers two families and a first region covering the whole key space. Each `put` is routed via `hbase:meta`, written to the server's write-ahead log and memory, and acknowledged. `get` fetches the row by key in milliseconds; `scan` walks sorted keys from `u0040` to before `u0050`. Note there is no `WHERE city='Pune'` — without a secondary index that would be a full scan.

Java client sketch (HBase 1.x style): `Put put = new Put(row); put.add(family, qualifier, value); table.put(put);` then `table.get(new Get(row))` and `table.getScanner(new Scan())`.

### 5. Under the hood
- Row-key order drives everything: sequential keys (timestamps, auto-increment IDs) send all writes to the *last* region ("hotspotting"). Fixes: salting (prefix with hash bucket), reversing keys, or hashing.
- Updates are not in place: a new version is appended; deletes write *tombstones*; background **compactions** merge files and drop old versions.
- Row-level operations are atomic; there are no multi-row transactions (in the classic design).
- It runs on any Hadoop filesystem, but by default (unconfigured) writes to the local disk — the first production setting is pointing `hbase.rootdir` at HDFS.

### 6. Performance and design implications
- Latency: single-digit milliseconds for key lookups; throughput scales by adding RegionServers.
- Design the **row key first** — it determines query capability and load distribution. Put the most-queried attribute first and avoid monotonically increasing prefixes.
- Keep column families few (1–3); each has its own memstore and files.
- Scans with filters still read data; for analytics export to Hive/Spark or query via Phoenix (SQL on HBase) or Hive-HBase integration.
- `count 'table'` in the shell scans everything; use the distributed `RowCounter` MapReduce job for big tables.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `create 't','cf'` | Make a table with family `cf` | Setup |
| `put` / `get` / `scan` / `delete` | CRUD by key | Everyday access |
| `list`, `describe`, `status` | Inspect tables, cluster | Operations |
| `disable` then `drop` | Remove a table | Cleanup |
| `hbase.rootdir` | Where data lives (HDFS URI) | Cluster setup |
| `VERSIONS`, `TTL`, `COMPRESSION` | Per-family options | Retention/size tuning |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Huge tables, key-based random read/write, high write rates | You need joins, ad-hoc queries, multi-row ACID |
| Sparse, evolving columns | Small data that fits an RDBMS |
| Time-series or messages keyed by entity + time | Access paths vary widely (no secondary indexes) |
| Real-time serving next to batch analytics on the same HDFS | You have no ops capacity: it's a complex system |

### 9. Common mistakes
- Monotonic row keys → one hot region → salt or reverse the key.
- Dozens of column families → many small flushes → keep to a few.
- Treating it like an RDBMS and scanning for non-key predicates → full-table scans → design key around the query or index elsewhere.
- Leaving default local filesystem → data lost with the machine → set `hbase.rootdir` to HDFS.

### 10. 🎯 Interview kit
**30-second answer:** HBase is a Bigtable-style distributed sorted map on HDFS: tables split into regions by row-key range, served by RegionServers, with column families, versioned cells and fast random reads/writes by key.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *HBase vs HDFS/Hive?* → random access vs scan-oriented batch.
- *How does a client find a row?* → ZooKeeper → `hbase:meta` → RegionServer.
- *How do you avoid hotspots?* → salting/hashing/reversal.
- *Is HBase column-oriented?* → no; column-family, row-oriented within family.

**Signals interviewers look for:** row-key design reasoning and honest limits (no joins, single index).

### 11. Check yourself
- [ ] Describe the data model as a nested map.
- [ ] Walk through a client `Get` without involving the master.
- [ ] Redesign a timestamp-keyed table to avoid hotspotting.
- [ ] Explain why "wide-column" is not "columnar".
<!-- explainer:end -->

## In this section (study in order)

1. [HBase Architecture and Schema Design](<HBase_Architecture/_index.md>) (L5 Expert)

## References

- *Hadoop: The Definitive Guide* — Tom White — HBase › Regions
- *Database Internals* — Alex Petrov — Introduction and Overview › Wide Column Stores
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Column-oriented storage and column families
- *Hadoop: The Definitive Guide* — Tom White — HBase › Loading Data
- *Hadoop: The Definitive Guide* — Tom White — HBase › Whirlwind Tour of the Data Model
- *Hadoop: The Definitive Guide* — Tom White — HBase › Tip
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Wide-column
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › GCP-Specific Networking and Multiregional Redundancy
- *Hadoop: The Definitive Guide* — Tom White — HBase › Note
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Regions

