---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: ["MySQL"]
---

# Composite vs Surrogate Keys

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A *composite key* uses several natural columns to identify a row; a *surrogate key* is a meaningless single-column identifier you add — choose by key width, stability, join cost and what the key's physical order does to the clustered index.

### 1. The big picture
Every table needs a way to identify rows. Two families exist:

- **Natural/composite key:** identity comes from business data — `(OrderID, LineNumber)`, `(StudentID, CourseID, Term)`.
- **Surrogate key:** the system generates a value with no business meaning — `IDENTITY`, `SEQUENCE`, `UUID` — and uses it as the primary key; natural keys are enforced separately with `UNIQUE` constraints.

The choice affects storage size (the key is copied into every nonclustered index and every child table's foreign key), join performance, insert patterns on the B-tree, how robust you are to business change, and how readable queries are. It is a design decision with performance consequences, not a religious one.

### 2. Core concepts and vocabulary
- **Natural key** — attribute(s) with business meaning that identify an entity (ISBN, email, country code).
- **Composite key** — a key made of two or more columns.
- **Surrogate key** — system-generated identifier without business meaning.
- **Candidate key / alternate key** — any minimal unique column set; the non-chosen ones become `UNIQUE` constraints.
- **Clustering key** — in SQL Server/InnoDB the column(s) that physically order the table; ideally *narrow, unique, static, ever-increasing*.
- **Identity / sequence** — monotonically increasing integer generator.
- **GUID/UUID** — 16-byte globally unique identifier; random (v4) or time-ordered (v7, `NEWSEQUENTIALID()`).
- **Key propagation** — child tables copy the parent key as FK; wide parent key = wide children and wide indexes.
- **Partition key / row key (NoSQL)** — key design that determines data distribution and lookup efficiency.

### 3. How it works, step by step
**What a key costs, mechanically:**
1. The PK (if clustered) defines row order; each row's physical location is its key.
2. Every nonclustered index stores the clustering key in each leaf entry as the row locator → key width multiplies across all indexes.
3. Every child table stores the full parent key as FK, and its own indexes carry it.
4. Joins compare key values: one `int` compare vs several column compares.
5. Inserts go where the key sorts: ever-increasing keys append to the end; random keys hit random pages → page splits, fragmentation, cache churn.

| Property | Composite natural key | Surrogate key |
|---|---|---|
| Width | Often wide (sum of columns) | Narrow (4–8 bytes int) |
| Stability | Business values change (rename, merger) → cascade updates | Never changes |
| Join syntax | Multi-column `ON` | One column |
| Self-describing | Yes — readable | No — need lookup |
| Natural uniqueness enforced? | By PK itself | Must add `UNIQUE` |
| Insert pattern | Depends on columns; may be random | Sequential if identity |
| Good for | Junction/association tables, narrow natural codes | Entities with wide/changing natural keys |

### 4. Worked example
```sql
-- Association (junction) table: composite PK is natural and sufficient
CREATE TABLE Enrollment (
    StudentID INT NOT NULL REFERENCES Student(StudentID),
    CourseID  INT NOT NULL REFERENCES Course(CourseID),
    Term      CHAR(6) NOT NULL,
    Grade     CHAR(2) NULL,
    CONSTRAINT PK_Enrollment PRIMARY KEY (StudentID, CourseID, Term)
);

-- Entity with a wide/unstable natural key: surrogate PK + unique natural key
CREATE TABLE Customer (
    CustomerID   INT IDENTITY(1,1) NOT NULL,
    TaxNumber    VARCHAR(20)  NOT NULL,
    Country      CHAR(2)      NOT NULL,
    Name         NVARCHAR(200) NOT NULL,
    CONSTRAINT PK_Customer PRIMARY KEY CLUSTERED (CustomerID),
    CONSTRAINT UQ_Customer_Natural UNIQUE (Country, TaxNumber)
);

-- Small code table: a narrow natural key IS a good key
CREATE TABLE OrderStatus (
    StatusCode CHAR(1) NOT NULL PRIMARY KEY,   -- 'O' open, 'C' closed
    StatusName VARCHAR(20) NOT NULL
);
```
**Check the cost:** compare `sp_spaceused` and index sizes for the same table keyed by `(Country, TaxNumber)` (≈ 22 bytes) vs `INT` (4 bytes) with five nonclustered indexes and three child tables — the wide key adds ~18 bytes × rows × (indexes + child tables). On 100 M rows that is gigabytes of extra RAM/IO. Also watch `sys.dm_db_index_physical_stats` fragmentation after random-GUID inserts vs identity inserts.

### 5. Under the hood
- **B-tree insert behavior.** With a sequential key every insert lands on the rightmost leaf; splits are rare and pages stay nearly full (some engines optimize this "right-only append" path). With random keys (GUID v4, hashed values), each insert goes to a random leaf; when full it splits ~50/50, leaving half-empty pages, higher write amplification, and a larger working set. Sequential GUIDs or time-ordered UUIDs restore locality.
- **Last-page contention.** The flip side: heavy concurrent inserts into an identity-ordered index can contend on the last page (latch `PAGELATCH_EX`). Remedies: partitioning, `OPTIMIZE_FOR_SEQUENTIAL_KEY`, or a hash-prefixed key — at the price of locality.
- **Clustered vs nonclustered PK.** The PK need not be the clustering key. A table can have a surrogate nonclustered PK for FKs and a clustered index on a business/range column (e.g. `(ReadAt, SensorID)` for time-series). Remember the clustering key is carried by all nonclustered indexes.
- **Composite key order.** Column order in a composite index decides which prefix searches use seeks: `(StudentID, CourseID, Term)` serves `WHERE StudentID = ?` and `StudentID = ? AND CourseID = ?`, not `Term = ?` alone — add a second index for other access paths.
- **NoSQL analogy.** In wide-column/key-value stores the composite is explicit: *partition key* picks the node, *row/sort key* orders within the partition (`CityName` + `SensorID_Timestamp`). Efficient queries use the partition key and a prefix/range of the sort key.
- **Pipelines.** Idempotent loads often build a composite "business key" from natural key + event time (or a hash of them) to deduplicate; the hash is a surrogate derived deterministically — not an identity.

### 6. Performance and design implications
- Keep clustering keys narrow, unique, static, ever-increasing (identity/sequence, or time-ordered).
- Use narrow natural keys when they exist (`CHAR(1)` status, `CHAR(2)` state) — they remove joins to lookup tables and are readable.
- For association tables, the composite of the two FKs (plus a discriminator like time) is usually the right PK and avoids an unnecessary extra index.
- Always enforce the natural key with `UNIQUE` when using a surrogate, otherwise duplicates silently appear.
- Child tables referencing a composite parent key copy every column — a three-column key means three FK columns everywhere.
- Don't expose surrogate integers in public APIs if enumeration is a risk; use a separate public identifier.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `IDENTITY(seed,inc)` / `SEQUENCE` | Generate surrogate values | Surrogate PKs |
| `NEWSEQUENTIALID()` / UUIDv7 | Time-ordered GUID | Distributed IDs with locality |
| `PRIMARY KEY NONCLUSTERED` + `CREATE CLUSTERED INDEX` | Separate PK from clustering | Range-scan-driven tables |
| `UNIQUE (cols)` | Enforce natural key | With every surrogate PK |
| `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` | Reduce last-page contention | High-rate identity inserts |
| `sys.dm_db_index_physical_stats` | Fragmentation/page density | After random-key inserts |
| `ALTER INDEX … REBUILD WITH (FILLFACTOR=…)` | Leave room for random inserts | Unavoidable random keys |

### 8. Trade-offs and decisions
| Choose composite/natural when | Choose surrogate when |
|---|---|
| Junction/association table | Natural key is wide, changeable, or composite across 3+ columns |
| Natural code is narrow and stable (ISO country, status code) | Key may be reused or corrected (names, phone numbers) |
| Data is partitioned/queried by those columns anyway | Many child tables reference the entity |
| You want self-describing data in reports | ORMs/frameworks expect single-column keys |
| Integration across systems uses the natural key | Sources may deliver conflicting natural keys (needs mapping layer) |

### 9. Common mistakes
- Surrogate key without a natural `UNIQUE` → duplicate entities → add the constraint.
- Wide composite clustered key → bloated indexes and FKs → surrogate or narrower key.
- Random GUID clustered PK → fragmentation, slow inserts → sequential GUID, bigint, or non-clustered PK.
- Using mutable business data as a PK → cascading updates → surrogate.
- Putting the low-selectivity column first in composite indexes without reason → poor seeks → order by query patterns and selectivity.
- Adding a surrogate to a junction table "for consistency" → extra index and no benefit → composite PK.

### 10. 🎯 Interview kit
**30-second answer:** A composite key identifies rows by several business columns; a surrogate is a generated, meaningless single-column ID. I use composite keys for association tables and narrow stable codes, and surrogates for entities with wide or changing natural keys — always enforcing the natural key with a unique constraint, and keeping the clustering key narrow, unique, static and increasing.

**Structure a longer answer:** clarify (usage, volume, who references the table) → mechanism (key copied into indexes/children, insert order) → trade-offs (width, stability, readability) → scenario (orders vs enrollments vs events).

**Likely follow-ups:**
- *Are GUIDs good keys?* → globally unique and generated anywhere, but wide and random; use sequential/time-ordered variants.
- *Does the PK have to be clustered?* → no; choose the clustering key for range scans/locality.
- *Composite index column order?* → equality columns first, matching query prefixes.
- *How do you deduplicate in a pipeline?* → deterministic business key (natural key + event time) with upsert.
- *Surrogate vs natural in a warehouse?* → surrogate keys in dimensions support SCD history.

**Signals interviewers look for:** you reason about physical impact (index width, splits), not just theory, and you keep integrity via unique constraints.

### 11. Check yourself
- [ ] Explain how clustering key width affects nonclustered index size.
- [ ] Design keys for `Student`, `Course`, `Enrollment` and justify each.
- [ ] Describe what happens on insert with random vs sequential keys.
- [ ] Say why a surrogate PK still needs a `UNIQUE` natural key.
- [ ] Order a composite index for three given query patterns.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What is a surrogate key in a dimension table and why is it preferred over the business key?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22550_What_is_a_surrogate_key_in_a_dimension_table_and_why_is_it_p.md>)
- [Basic] [What is PII in a dataset and how would a data engineer mask or hash sensitive columns before sharing a table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22641_What_is_PII_in_a_dataset_and_how_would_a_data_engineer_mask_.md>)
- [Basic] [What is a candidate key, super key, and composite key?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22188_What_is_a_candidate_key_super_key_and_composite_key.md>)
- [Intermediate] [What is a natural key versus a surrogate key, and what problems arise from using a natural key?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22297_What_is_a_natural_key_versus_a_surrogate_key_and_what_proble.md>)
- [Deep] [What is a surrogate key pipeline / key mapping table and why is it needed when integrating data from multiple sources?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22580_What_is_a_surrogate_key_pipeline_key_mapping_table_and_why_i.md>)
- [Deep] [What is data skew in a distributed query and why can one heavy key make a join slow on a cluster?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22621_What_is_data_skew_in_a_distributed_query_and_why_can_one_hea.md>)
- [Expert] [Why can random UUIDv4 primary keys hurt insert performance, and how does UUIDv7 or ULID help?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22299_Why_can_random_UUIDv4_primary_keys_hurt_insert_performance_a.md>)
- [Expert] [What is a shard key and what happens if you pick a low-cardinality or monotonically increasing one?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22322_What_is_a_shard_key_and_what_happens_if_you_pick_a_low-cardi.md>)

## Related concepts

- [Primary and Unique Keys](<../../Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>) *(Constraints and Keys)*
- [Surrogate Keys in Normalization](<../../Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>) *(Normalization)*
- [Surrogate vs Natural Keys for Lookups](<../../Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>) *(Schema Design)*

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Surrogate Key
- *Azure Data Fundamentals* — Michael John Pena — Azure Storage Solutions › Row keys: Ensuring uniqueness
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Concrete example

