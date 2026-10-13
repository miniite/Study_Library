---
id: "02_Data_Engineering/02_Storage_and_Formats/Data_Models_and_Query_Languages"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Data Models and Query Languages

**L4 Advanced** · [↑ Storage Systems and File Formats](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A data model decides what you can express and how easily — relational, document and graph models each fit different relationship shapes, and declarative query languages (SQL, Cypher, SPARQL, Datalog) let the engine choose how to execute, which is why they dominate both databases and modern batch frameworks.

### 1. The big picture

Applications are built from layers of data models: real-world entities become objects in code; objects are stored in a general-purpose model (tables, documents, graph); the database maps that model to bytes on disk; hardware maps bytes to physical signals. Each layer hides the one below it. The middle layer — the general-purpose data model — has an outsized effect, because it decides what is easy, what is awkward and what performs badly.

For data engineers this matters twice. Source systems arrive in different models (relational tables, JSON documents, graph data), and your analytical platform must reconcile them. And the processing tools you use — SQL warehouses, Spark, Hive, Flink — are all descendants of the question "should the user say *what* they want or *how* to compute it?" This lesson compares the three main models and the query languages that go with them, then connects to how batch frameworks adopted declarative ideas.

### 2. Core concepts and vocabulary

- **Relational model** — data as relations (tables) of unordered tuples (rows); introduced by Codd in 1970.
- **Document model** — self-contained documents (JSON/XML) holding nested one-to-many structures.
- **Graph model** — **vertices** (entities) and **edges** (relationships); property graphs and triple stores.
- **Impedance mismatch** — awkward translation between application objects and relational tables; ORMs reduce but don't remove it.
- **Shredding** — splitting a document-like structure into several relational tables.
- **One-to-many / many-to-one / many-to-many** — relationship cardinalities that steer model choice.
- **Schema-on-write vs schema-on-read** — enforced at storage time vs interpreted at read time.
- **Declarative vs imperative query** — state the result pattern vs spell out the steps.
- **Query optimiser** — component choosing indexes, join order and algorithms.
- **Cypher / SPARQL / Datalog** — declarative graph query languages (property graph, triple store/RDF, rule-based).
- **Recursive CTE** — SQL `WITH RECURSIVE` for variable-length traversal.
- **MPP database** — massively parallel processing database with tightly integrated storage, planner and executor.

### 3. How it works, step by step

How to choose a model for a dataset:

1. **List the entities and relationships.** Draw them: tree-shaped, flat, or a web?
2. **Classify the relationships.**
   - Mostly one-to-many, loaded as a unit → document.
   - Many-to-one and many-to-many, ad hoc joins, strong integrity → relational.
   - Highly interconnected, variable-depth traversals → graph.
3. **Consider the access pattern.** Whole-record fetch benefits from document locality; analytical aggregation benefits from relational/columnar structure; path queries need graph traversal.
4. **Consider schema stability.** Externally controlled, changing shapes favour schema-on-read; stable shared structures favour schema-on-write.
5. **Pick the query language** that the engine can optimise and your team can maintain.

| | Relational | Document | Graph |
|---|---|---|---|
| Unit | Row in table | Nested document | Vertex + edges |
| Strength | Joins, many-to-many, integrity, ad hoc queries | Locality, flexible schema, matches app objects | Variable-depth relationships, heterogeneous data |
| Weakness | Object mismatch, shredding of trees | Weak joins, can't reference nested items directly | Less mature tooling for aggregation at scale |
| Typical query | SQL | Query API / aggregation pipeline / SQL-ish | Cypher, SPARQL, Datalog |

### 4. Worked example

The same question — "people who were born in the US and now live in Europe" — in a declarative graph language and in SQL.

```text
-- Cypher (property graph): variable-length WITHIN traversal is one token
MATCH (person) -[:BORN_IN]->  () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
      (person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'})
RETURN person.name
```

```sql
-- SQL (graph stored as vertices/edges tables) needs recursive CTEs
WITH RECURSIVE
  in_usa(vertex_id) AS (
      SELECT vertex_id FROM vertices WHERE properties->>'name' = 'United States'
    UNION
      SELECT e.tail_vertex FROM edges e JOIN in_usa u ON e.head_vertex = u.vertex_id
      WHERE e.label = 'within'),
  in_europe(vertex_id) AS (
      SELECT vertex_id FROM vertices WHERE properties->>'name' = 'Europe'
    UNION
      SELECT e.tail_vertex FROM edges e JOIN in_europe u ON e.head_vertex = u.vertex_id
      WHERE e.label = 'within'),
  born_in_usa(vertex_id)       AS (SELECT e.tail_vertex FROM edges e JOIN in_usa    u ON e.head_vertex = u.vertex_id WHERE e.label = 'born_in'),
  lives_in_europe(vertex_id)   AS (SELECT e.tail_vertex FROM edges e JOIN in_europe u ON e.head_vertex = u.vertex_id WHERE e.label = 'lives_in')
SELECT v.properties->>'name'
FROM   vertices v
JOIN   born_in_usa b       ON b.vertex_id = v.vertex_id
JOIN   lives_in_europe l   ON l.vertex_id = v.vertex_id;
```

Reading the result: both return the same people, but the SQL version needs four auxiliary sets because the number of hops (street → city → region → state → country) is not known in advance, whereas `WITHIN*0..` means "follow zero or more `within` edges". Datalog states the same logic as rules (`within_recursive` defined in terms of itself), built up one rule at a time.

### 5. Under the hood

**Why relational won.** The relational model laid data out "in the open": any row is accessible by conditions, with no nested access paths for the programmer to follow. The query optimiser chooses the access path (indexes, join order) automatically, so adding an index speeds queries without rewriting them. The optimiser is expensive to build once but benefits every application — a general-purpose solution that beat hand-coded access paths of earlier hierarchical and network (CODASYL-style) models, many of whose ideas reappear in modern document stores.

**Documents vs relations today.** Documents win on schema flexibility, locality (one read fetches the whole tree) and closeness to application objects. Relations win on joins and many-to-one/many-to-many support. Document databases that lack good joins push the work to application code (several round trips or denormalisation with consistency burden). Modern relational systems also store JSON columns, while some document systems add join-like operators; the models are converging. "Schemaless" really means **schema-on-read**: the structure is implicit and the application interprets it, analogous to dynamic typing; it is a good fit when the structure is outside your control and changes at any time.

**Declarative advantages.** Because the user does not prescribe order or algorithm, the engine can reorder, choose indexes, move records or run in parallel without breaking queries. The imperative alternative locks in assumptions the engine can't verify (for instance, reliance on record order). Declarative languages also suit multi-core execution since nothing in the query forces sequential steps.

**Graph models.** In a property graph each vertex and edge has an ID, a label and properties; Cypher pattern-matches over them. Triple stores model everything as (subject, predicate, object) and use SPARQL; Datalog generalises to `predicate(subject, object)` with rules, reusable like functions and recursion. All can be encoded in relational tables with joins, but variable-length traversal becomes cumbersome.

**Batch frameworks converge on declarative.** MPP databases are monolithic: the storage layout, planner and executor are tuned together, which gives excellent performance and SQL accessibility, but not every workload (ML features, full-text ranking, image analysis) fits SQL. MapReduce let engineers run arbitrary code over HDFS data; on top of that Hive built SQL. Later dataflow engines (Spark, Flink, Hive, Impala) added cost-based optimisers and declarative APIs — letting them pick join strategies, read only needed columns and use vectorised execution — while retaining the ability to call arbitrary user code. The result: batch frameworks start to resemble MPP databases in performance and keep a flexibility advantage.

### 6. Performance and design implications

- **Join cost**: many-to-many in document stores means application-side joins; plan denormalisation deliberately.
- **Locality**: documents are read whole; large unbounded arrays inside documents cause hot, oversized records.
- **Traversal depth**: relational joins are fixed by the query; deep or variable-depth traversals perform and read better in graph engines or with recursive CTEs on small graphs.
- **Optimiser reliance**: write declarative code and keep statistics fresh; rely on the engine's plan instead of manual ordering.
- **UDF overhead**: user-defined callbacks per record are slow compared with declarative filters/projections that enable column pruning and vectorisation.
- **Measure**: query plans (`EXPLAIN`), rows examined vs returned, number of round trips per request, traversal depth distribution.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN` / `EXPLAIN ANALYZE` | Shows optimiser's plan and actual costs | Verifying that queries use indexes and good joins |
| `WITH RECURSIVE …` | Variable-length traversal in SQL | Hierarchies, small graphs |
| `JSON`/`JSONB` column + path operators (`->>`) | Stores and queries semi-structured data in relational DBs | Hybrid models |
| `CREATE INDEX` | Adds an access path without changing queries | Speeding relational queries |
| `MATCH … RETURN` (Cypher) | Pattern-matches in a property graph | Relationship-heavy analysis |
| `SELECT … WHERE … (SPARQL)` | Queries triple stores | Linked data, ontologies |
| Spark `df.explain()` / Hive `EXPLAIN` | Shows logical/physical plan of batch queries | Tuning join strategies |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Relational — structured data, many relationships, strong integrity, ad hoc analytics | Rapidly changing, deeply nested structures loaded as units |
| Document — tree-shaped aggregates, flexible schema, locality | Many-to-many relationships dominate |
| Graph — social, fraud, network, knowledge graphs; variable-depth queries | Simple tabular aggregation at massive scale |
| Declarative SQL — analytics, optimiser can help | Logic that needs custom libraries (ML, NLP) → mix with code |
| MapReduce/Spark UDFs — arbitrary processing | Simple filters and projections that should be declarative |
| MPP database — tuned SQL performance, BI tools | Heterogeneous, non-SQL workloads on shared files |

### 9. Common mistakes

- **Shredding a document-shaped object into many tables** → complicated schema and code → keep aggregates together (JSON column or document store) when always read as a unit.
- **Using a document store for heavily many-to-many data** → application-level joins and inconsistency → relational or graph model.
- **Calling "schemaless" free** → implicit schema scattered through code → document the schema and validate on read.
- **Hand-optimising query order** → plan changes make it wrong → trust the optimiser, keep statistics fresh, check `EXPLAIN`.
- **Writing per-record UDFs for simple filters** → CPU overhead and no pruning → use declarative expressions.
- **Forcing graph problems into fixed-depth joins** → incomplete answers when depth varies → recursive CTE or graph engine.
- **Importing JSON as an opaque text column** → no querying or indexing inside → use native JSON types.

### 10. 🎯 Interview kit

**30-second answer:** "Relational models organise data into tables and are strong at joins and ad hoc queries; document models store nested aggregates with flexible schema and locality; graph models are for highly connected data with variable-depth traversals. Declarative languages like SQL let the optimiser choose execution, which is why even Spark and Hive adopted them — while still allowing arbitrary code when needed."

**Structure a longer answer:** clarify relationships and access patterns → compare models → query language implications → trade-offs → scenario (e.g., fraud rings in a graph, orders in relational, user profiles in documents).

**Likely follow-ups:**
- *What is the object-relational impedance mismatch?* → Tables/rows vs nested objects; ORMs help partially.
- *Why does SQL being declarative matter?* → Engine can optimise and parallelise; queries survive engine changes.
- *When would you pick a graph database?* → Relationship-centric queries with unknown depth (recommendations, fraud, lineage).
- *Is schemaless really schemaless?* → No, schema-on-read; the application assumes a structure.
- *How do Spark and Hive relate to declarative queries?* → Cost-based optimisers choose join algorithms; column pruning and vectorisation apply to declarative operations.

**Signals interviewers look for:** reasoning from relationship shape, awareness of convergence between models, knowing the cost of application-level joins.

### 11. Check yourself

- [ ] Explain the object-relational mismatch with a résumé example.
- [ ] Compare document and relational models for one-to-many vs many-to-many data.
- [ ] Explain why declarative queries enable optimisation and parallelism.
- [ ] Write a recursive CTE that finds all locations within a region.
- [ ] Describe the difference between a property graph and a triple store.
- [ ] Explain how MapReduce, Hive and Spark moved toward declarative processing.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Graphs and Iterative Processing › The move toward declarative query languages
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › The Object-Relational Mismatch
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › Query Languages for Data
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › Relational Model Versus Document Model
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › Graph-Like Data Models
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › Chapter 2. Data Models and Query Languages
- *Designing Data-Intensive Applications* — Martin Kleppmann — Comparing Hadoop to Distributed Databases › Diversity of processing models
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › Relational Versus Document Databases Today
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › Graph Queries in SQL
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › The Foundation: Datalog

