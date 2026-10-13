---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SQL_vs_NoSQL"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["MySQL", "Oracle", "PostgreSQL", "AWS", "SQL_Server"]
---

# SQL vs NoSQL

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL is a *language* (and relational is a *model*); "NoSQL" is a loose label for stores that trade some relational guarantees (joins, strict schema, multi-row transactions) for scale-out, flexible shape, or specialized access patterns, and the right choice comes from your data shape, access patterns and consistency needs, not slogans.

### 1. The big picture
Two separate questions get mixed up:

1. **What is the language/model?** SQL is the standard declarative language for querying and changing data. The relational model organizes data into tables of rows with typed columns, keys and constraints. A **DBMS** such as PostgreSQL, MySQL, SQL Server or Oracle implements the model and the language. So "SQL vs MySQL" is a category error: SQL is the language, MySQL is one product that speaks a dialect of it.
2. **How is the data stored and accessed?** "NoSQL" (originally "not only SQL") covers key-value stores, document stores, wide-column stores, graph databases, and often time-series and search engines. Many now support SQL-like query languages or even transactions; the label describes a *design emphasis*, not a technology.

Relational systems give you a declarative query optimizer, joins, constraints and ACID transactions over multiple rows, which is wonderful when the data is interrelated and queries are unpredictable. NoSQL systems typically optimize for one access pattern (get by key, get a whole aggregate document, traverse edges, append events) and for horizontal scale or availability.

### 2. Core concepts and vocabulary
- **Relational model** — data as relations (tables); relationships via keys; operations from relational algebra.
- **Schema-on-write** — structure is enforced when data is stored (relational). **Schema-on-read** — structure is interpreted by the reader (many document stores, data lakes).
- **ACID** — Atomicity, Consistency, Isolation, Durability for transactions.
- **BASE** — Basically Available, Soft state, Eventually consistent: shorthand for relaxed guarantees.
- **Aggregate** — a bundle of data read/written together (an order with its lines), the unit in document stores.
- **Horizontal scaling (sharding)** — spread data across many nodes by a key.
- **CAP / PACELC** — during a network partition choose consistency or availability; otherwise latency vs consistency.
- **Polyglot persistence** — using several database types, each for what it is good at.
- **OLTP / OLAP** — many small transactions vs few large analytical scans.

### 3. How it works, step by step
The families and what each optimizes:

| Family | Data model | Optimized for | Examples |
|---|---|---|---|
| Relational | Tables, rows, foreign keys | Ad hoc queries, joins, integrity, transactions | PostgreSQL, MySQL, SQL Server, Oracle |
| Key-value | Opaque value by key | Very fast point get/put, caching, sessions | Redis, DynamoDB (also document-like) |
| Document | JSON-like documents | Reading/writing a whole aggregate, evolving shape | MongoDB, Couchbase |
| Wide-column | Rows with sparse column families, partition key + clustering key | Huge write volume, time-ordered partitions, known queries | Cassandra, HBase |
| Graph | Nodes and edges | Multi-hop traversals | Neo4j, SQL Graph |
| Time-series | Timestamped points | Append and time-window aggregates | InfluxDB, TimescaleDB |
| Search | Inverted index | Full-text relevance | Elasticsearch, OpenSearch |

A decision walk-through:
1. **List the entities and relationships.** Highly interrelated, queried in many ways → relational.
2. **List the access patterns.** If you can write them all down in advance and each is "get by key / range within a partition", a key-value or wide-column design becomes viable.
3. **State the consistency need.** Money, inventory, uniqueness → strong consistency and transactions.
4. **Estimate scale.** A single well-indexed relational server handles terabytes and tens of thousands of queries per second; sharding is needed later than most teams think.
5. **Pick the simplest thing** that meets 1–4, and add a second store only for a clear gap (cache, search, analytics).

### 4. Worked example
Feature: a shopping cart and orders.

Relational design:

```sql
CREATE TABLE customers (customer_id bigint PRIMARY KEY, email text UNIQUE NOT NULL);
CREATE TABLE orders (
  order_id    bigint PRIMARY KEY,
  customer_id bigint NOT NULL REFERENCES customers,
  placed_at   timestamptz NOT NULL,
  total_cents bigint NOT NULL CHECK (total_cents >= 0)
);
CREATE TABLE order_lines (
  order_id   bigint REFERENCES orders,
  line_no    int,
  sku        text   NOT NULL,
  qty        int    NOT NULL CHECK (qty > 0),
  price_cents bigint NOT NULL,
  PRIMARY KEY (order_id, line_no)
);

-- Ad hoc question nobody planned for:
SELECT c.email, SUM(l.qty * l.price_cents) AS revenue_cents
FROM   customers c JOIN orders o USING (customer_id)
       JOIN order_lines l USING (order_id)
WHERE  o.placed_at >= now() - interval '30 days'
GROUP  BY c.email ORDER BY revenue_cents DESC LIMIT 10;
```

Document design (same order as one document):

```json
{ "_id": 9001, "customer": {"id": 7, "email": "a@x.com"},
  "placedAt": "2026-10-01T10:00:00Z",
  "lines": [ {"sku": "A1", "qty": 2, "priceCents": 1500} ] }
```

Reading one order = one lookup, no join. But "top customers by revenue last 30 days" needs an aggregation pipeline scanning many documents, the customer email is duplicated into every order (update anomalies), and keeping stock counts consistent across documents needs careful design. The relational model wins here because the questions vary and integrity matters; the document model would win for a content catalogue where each item is self-contained and attributes differ per item.

### 5. Under the hood
- **Why relational joins scale vertically well:** B-tree indexes, a cost-based optimizer that chooses join order and algorithm (nested loop, hash, merge), and buffer caching make multi-table queries efficient on one node. Distributing the same joins across nodes is expensive because rows must be shuffled over the network.
- **Why many NoSQL systems partition by key:** hashing the partition key spreads data and load evenly; a query that includes the key goes to one node. A query without it must scatter-gather. That is why the data model is designed *from the queries* (query-first modelling) rather than from the entities.
- **Consistency:** leaderless/quorum systems (Cassandra, Dynamo-style) tune read/write quorums (`R + W > N` gives overlap). Single-leader relational systems give strong consistency on the leader and eventual on replicas.
- **Convergence:** relational products now add JSON columns (`jsonb`), and document stores add multi-document transactions; the line is blurry. "NewSQL" systems (CockroachDB, Spanner, TiDB) pair SQL with distributed transactions.

### 6. Performance and design implications
- Relational: add indexes for access paths, normalize for integrity, denormalize deliberately for read hot spots; use partitioning before sharding.
- Document: embed data that is read together and changes together; reference data that is shared or large; beware unbounded arrays.
- Key-value/wide-column: choose partition keys to avoid hot partitions and unbounded partition size.
- Always benchmark with your data shape; micro-benchmarks of "reads per second" hide the cost of the queries you can't express.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `jsonb` column + GIN index (PostgreSQL) | Semi-structured data inside a relational DB | Variable attributes without leaving SQL |
| `CHECK`, `FOREIGN KEY`, `UNIQUE` | Enforce integrity in the DB | When correctness matters more than flexibility |
| Partitioning (`PARTITION BY`) | Split a table within one DB | Large time-series tables |
| Read replicas | Scale reads | Read-heavy OLTP |
| Sharding key choice | Distribute rows over nodes | When one node can't hold writes |
| Quorum settings (`R`, `W`) | Trade latency for consistency in leaderless stores | Dynamo-style systems |

### 8. Trade-offs and decisions
| Choose relational when | Choose a NoSQL family when |
|---|---|
| Data is interrelated; many ad hoc queries; constraints and transactions needed | Access patterns are few and known; a single aggregate is the unit of work |
| Reporting and analytics on the same data are required | Write volume or dataset size exceeds a single node and sharding is unavoidable |
| Team knows SQL; tooling and ecosystem matter | Data is schemaless/evolving rapidly, or is graph/time-series/search shaped |
| Strong consistency for money/inventory/uniqueness | Availability under partition matters more than immediate consistency |

### 9. Common mistakes
- Picking NoSQL "for scale" at a scale one PostgreSQL box handles → lost joins, constraints, transactions → start relational and extract hot paths later.
- Treating a document store as relational (many cross-document references) → application-side joins and no integrity → either embed or use a relational DB.
- Ignoring consistency semantics of replicas/quorums → stale reads, lost updates → read the guarantees, use conditional writes.
- Equating "SQL database" with "single server" → missing replication, partitioning, distributed SQL options.
- Using `jsonb` for everything → loses types, constraints, stats → promote stable fields to real columns.

### 10. 🎯 Interview kit
**30-second answer:** SQL is the query language; relational databases implement it with schema, joins, constraints and ACID. NoSQL covers key-value, document, wide-column, graph stores that give up some of that for flexible shape or horizontal scale. I choose by data relationships, query patterns and consistency needs, and I'd use polyglot persistence only where a second store clearly earns its operational cost.

**Structure a longer answer:** define terms → what each family optimizes → decision criteria (relationships, access patterns, consistency, scale, team) → apply to the scenario → how I'd scale the choice (replicas, partitioning, caching) and where I'd use the other kind.

**Likely follow-ups:**
- *Is NoSQL always faster?* → only for the access pattern it was designed for.
- *Does NoSQL mean no transactions?* → no; varies by product.
- *Explain CAP.* → under partition, choose C or A; PACELC adds latency/consistency without partition.
- *How would you scale the relational choice?* → indexes, replicas, partitioning, caching, then sharding.

**Signals interviewers look for:** reasoning from data shape and consistency, not brand names; an honest statement of where the alternative would be better.

### 11. Check yourself
- [ ] Explain "SQL vs MySQL" in one sentence.
- [ ] For three features (cart, activity feed, social graph) choose a store and justify.
- [ ] Describe what `R + W > N` guarantees and what it doesn't.
- [ ] Show how a document model creates an update anomaly.
- [ ] Name two ways relational databases absorbed NoSQL features.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [Oracle](<../../../TOOLS/Oracle/_index.md>)
- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)
- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [What is the difference between the SQL and MySQL, and is SQL a language or a database?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39757_What_is_the_difference_between_the_SQL_and_MySQL_and_is_SQL_.md>)
- [Basic] [What is the difference between DROP TABLE and DROP TABLE IF EXISTS, and why use the latter in a script?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39859_What_is_the_difference_between_DROP_TABLE_and_DROP_TABLE_IF_.md>)
- [Basic] [Explain the difference between DDL, DML, DCL, and TCL commands.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40487_Explain_the_difference_between_DDL_DML_DCL_and_TCL_commands.md>)
- [Basic] [What is the difference between OLTP and OLAP, and why do you extract from OLTP but query analytics against OLAP?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40035_What_is_the_difference_between_OLTP_and_OLAP_and_why_do_you_.md>)
- [Intermediate] [When would you drop down to raw SQL instead of using your ORM's query builder?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40271_When_would_you_drop_down_to_raw_SQL_instead_of_using_your_OR.md>)
- [Intermediate] [What is the difference between the TRUNCATE and DELETE commands in terms of rollback and triggers?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40808_What_is_the_difference_between_the_TRUNCATE_and_DELETE_comma.md>)
- [Deep] [Why did you choose a relational database over a NoSQL store for this feature? Walk me through the trade-off.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39778_Why_did_you_choose_a_relational_database_over_a_NoSQL_store_.md>)
- [Deep] [What is the difference between OLTP and OLAP workloads?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41097_What_is_the_difference_between_OLTP_and_OLAP_workloads.md>)

## Related concepts

- [Databases and Schemas](<../../Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>) *(DBMS Concepts)*
- [Views and Materialized Views](<../../Database_Internals/DBMS_Concepts/Views_and_Materialized_Views.md>) *(DBMS Concepts)*
- [Dynamic SQL](<../../Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 11. Stream Processing › Message brokers compared to databases
- *Designing Data-Intensive Applications* — Martin Kleppmann — Glossary
- *Hadoop: The Definitive Guide* — Tom White — Comparison with Databases

