---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems"
tools: ["Elasticsearch"]
---

# Search Index Internals and Mappings

**L4 Advanced** · [↑ Search Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A search engine (Lucene under Elasticsearch/OpenSearch) turns documents into immutable, sorted segments of inverted indexes and column-oriented doc values; your *mapping* decides which of those structures each field gets — and therefore what you can search, sort, aggregate and afford.

### 1. The big picture
A relational index maps *row → key*; an inverted index maps *term → list of documents*. That reversal is what makes "find every document containing these words" fast. Elasticsearch and OpenSearch (a managed variant is Amazon OpenSearch Service, built on Apache Lucene and used for log analytics, security analytics and full-text search) wrap Lucene in a distributed layer: an **index** is split into **shards**, each shard is a Lucene index, shards are replicated, and a coordinating node scatters a query and gathers results.

Everything starts with the **mapping** — the schema that says whether a string is analysed text or an exact keyword, whether a number is an integer or a float, whether a field is indexed, stored, or only retrievable from `_source`. Bad mappings are the most common cause of "why can't I sort on this?", "why is the cluster out of heap?" and "why do searches miss obvious documents?".

### 2. Core concepts and vocabulary
- **Document** — a JSON object; the unit of indexing and retrieval.
- **Index / shard / replica** — logical collection; its horizontal partitions (primary count fixed at creation); copies for HA and read throughput.
- **Segment** — an immutable Lucene file set; new documents create new segments, merged in the background.
- **Inverted index** — term dictionary + postings lists (doc ids, frequencies, positions).
- **Analyzer** — char filters → tokenizer → token filters (lowercase, stop words, stemming, synonyms) that turn text into terms, at index and query time.
- **`text` field** — analysed, searchable by words, no sorting/aggregations by default.
- **`keyword` field** — not analysed (exact), supports term queries, sorting, aggregations.
- **Doc values** — on-disk columnar structure used for sorting, aggregations and scripts.
- **Fielddata** — in-heap uninverted structure for `text` fields; avoid.
- **Multi-field** — one source field indexed several ways (`title` as `text` and `title.keyword`).
- **Refresh / flush / translog** — refresh makes new data searchable (default every 1 s); the translog gives durability between commits; flush does a Lucene commit.
- **BM25** — the default relevance function (see the relevance lesson).

### 3. How it works, step by step
**Indexing**
1. The coordinating node routes by `hash(_routing or _id) % number_of_primary_shards` to a primary shard.
2. The primary parses the document with the mapping, analyses text fields, builds in-memory segment buffers, and appends the operation to the **translog**.
3. It forwards the operation to replicas; the request succeeds when the required shard copies acknowledge.
4. On **refresh** (default each second) the buffer becomes a new searchable segment — this is why search is *near*-real-time.
5. On **flush** (or when the translog is large) segments are fsynced via a Lucene commit and the translog is trimmed.
6. **Merge** policy combines small segments into larger ones, dropping deleted docs. Updates are *delete + insert*.

**Searching**
1. The coordinator sends the query to one copy of each relevant shard (*query phase*): each shard returns top-N doc ids and scores.
2. The coordinator merges the lists and picks the global top-N, then asks the owning shards for the actual documents (*fetch phase*).
3. Aggregations run in the query phase on doc values and are merged by the coordinator.

| Need | Field type | Backed by |
|---|---|---|
| Full-text match on words | `text` | inverted index |
| Exact id, status, tag, sort/agg on string | `keyword` | inverted index + doc values |
| Numeric/date range | `integer`, `date`, … | BKD tree + doc values |
| Nearest-neighbour | `dense_vector`/`knn_vector` | HNSW graph |
| Not searchable, only returned | `enabled:false`/`index:false` | `_source` only |

### 4. Worked example
```json
PUT products
{
  "settings": { "number_of_shards": 3, "number_of_replicas": 1 },
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "title":  { "type": "text",
                  "fields": { "raw": { "type": "keyword", "ignore_above": 256 } } },
      "sku":    { "type": "keyword" },
      "price":  { "type": "scaled_float", "scaling_factor": 100 },
      "created":{ "type": "date" },
      "attrs":  { "type": "flattened" }
    }
  }
}
```
```json
GET products/_search
{
  "query": { "match": { "title": "wireless headphones" } },
  "sort":  [ { "title.raw": "asc" } ],
  "aggs":  { "by_brand": { "terms": { "field": "sku" } } }
}
```
Sorting on `title` (a `text` field) would fail or need fielddata; sorting on `title.raw` uses doc values. `dynamic: strict` rejects unknown fields rather than silently inventing mappings. `flattened` maps an object with unpredictable keys as one field to avoid a **mapping explosion**. Inspect with `GET products/_mapping`, `GET _analyze { "analyzer":"standard","text":"…" }` and `GET products/_stats`.

### 5. Under the hood
- **Segments are write-once.** Deletes are bits in a per-segment live-docs bitmap; space is reclaimed only on merge. Many tiny segments (very frequent refresh) slow search and cost file handles; `force_merge` on read-only indexes cuts both.
- **Term dictionary** is an FST (finite-state transducer) held off-heap by the OS file cache; postings are compressed delta-coded blocks with skip lists; positions enable phrase queries.
- **BM25** scores a term as `IDF · tf·(k1+1) / (tf + k1·(1 − b + b·|d|/avgdl))`; defaults `k1=1.2`, `b=0.75`. `k1` controls term-frequency saturation; `b` controls length normalisation (b=0 ignores length — right for short titles; high b penalises long documents). Scores are computed per shard using *shard-local* IDF unless you use `dfs_query_then_fetch`, which matters on tiny indexes.
- **Cardinality aggregation** uses HyperLogLog++ with a `precision_threshold` (default 3000, max 40,000): counts below the threshold are near-exact; above it you get ~1–6% error with fixed memory. Use exact counting (terms agg with a sort and paging, or a pre-aggregation) when billing or compliance needs exactness.
- **Cluster state.** Mappings live in cluster state shared by all nodes. Dynamic fields from many tenants bloat it; `index.mapping.total_fields.limit` (default 1000) is the safety valve.
- **Index sorting** (`index.sort.field`) stores segments in a chosen order so queries sorted by that field can terminate early; cost is slower indexing and merging.
- **Time-series data** is best in data streams with ILM, `_source` compression, and rollup/transform jobs when long-term granularity can be coarser.

### 6. Performance and design implications
- **Map only what you search.** If 8 of 50 fields are queried, set the rest `index:false` (or `enabled:false` for objects) and leave `doc_values:false` where never sorted/aggregated; use `dynamic: strict` or `false` and dynamic templates.
- **Shard count** is permanent for an index (only split/shrink APIs or reindex change it); keep shards in the tens of GB and avoid hundreds of tiny ones — each shard costs heap, file handles and cluster-state work.
- **`refresh_interval`**: raise to 30 s or `-1` during bulk loads; use `_bulk` with a few MB per request.
- **Use keyword for ids/enums**, not text; use `numeric`/`date` types — not strings — for ranges.
- **Diagnose slowness** with `index.search.slowlog.threshold.*`, `GET _nodes/hot_threads`, `GET _nodes/stats`, `_cat/shards`, `_cat/segments`, and the `profile` API. Typical culprits: expensive aggregations on high-cardinality fields, wildcard/regex queries, deep `from+size`, scripts, or heavy merges.
- **Backups.** Snapshots are not a validation of data integrity: corrupted shards replicate. Verify snapshots by restoring them periodically.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GET idx/_mapping`, `PUT idx/_mapping` | Inspect/add field mappings | After any schema change |
| `POST _reindex` | Copy into a new mapping | Changing a field type (can't change in place) |
| `GET _analyze` | Shows tokens | Debug "why didn't it match" |
| `index.refresh_interval` | Searchability delay vs throughput | Bulk loads |
| `index.mapping.total_fields.limit` | Cap on field count | Prevent mapping explosion |
| `POST idx/_forcemerge?max_num_segments=1` | Compact segments | Read-only/rolled-over indexes |
| `_cat/shards?v`, `_cluster/health` | Shard layout and state | Capacity and health |
| `PUT _snapshot/...` | Backups | DR plan |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `keyword` for ids, statuses, tags | You need word-level full-text search |
| `text` + `keyword` multi-field | Field is long (fielddata/doc values on huge strings waste disk) |
| `flattened` for free-form attributes | You must filter/aggregate by type or range on sub-keys |
| One index per time period (ILM) | Millions of tiny indexes (too many shards) |
| Index sorting | Write-heavy path where merge cost matters more |

### 9. Common mistakes
- Mapping IDs as `text` → term queries fail because of tokenisation → use `keyword`.
- Leaving dynamic mapping on → "mapping explosion", huge cluster state → strict/false plus templates.
- Over-sharding → master and heap pressure, slow cluster-state updates → shrink/reindex, consolidate by rollover.
- Sorting or aggregating on `text` → error or fielddata heap blow-up → use a keyword sub-field.
- Assuming search is real time → missing just-written docs → use `refresh=wait_for` sparingly or design for 1 s lag.
- Relying on a single snapshot → corrupted data restored → test restores.

### 10. 🎯 Interview kit
**30-second answer:** Lucene stores each shard as immutable segments with an inverted index for text, BKD trees for numerics and doc values for sort/aggregation. Mapping chooses which of those structures a field gets. Writes go to the translog and an in-memory buffer, become searchable on refresh (1 s), and are merged in the background.

**Structure a longer answer:** inverted index → analyzer → mapping types and multi-fields → write path (translog, refresh, flush, merge) → read path (query/fetch) → operational costs (shards, heap, cluster state).

**Likely follow-ups:**
- *text vs keyword?* → analysed vs exact; sort/agg break on text.
- *Why can't I change a field's type?* → existing segments are immutable; reindex.
- *HyperLogLog++ trade-off?* → fixed memory, ~1–6% error above the precision threshold.
- *k1 and b?* → saturation and length normalisation; wrong b shows as long docs always dominating or being buried.

**Signals interviewers look for:** you tie each feature to a data structure and its cost; you know mappings are largely immutable; you mention shard count and cluster state.

### 11. Check yourself
- [ ] Draw the path from `PUT doc` to "searchable".
- [ ] Explain why sorting on a `text` field is a problem and give two fixes.
- [ ] Choose settings for a 5 TB log index with 20 searched fields out of 400.
- [ ] Explain how a delete frees disk space.
- [ ] Say what `dfs_query_then_fetch` fixes.
<!-- explainer:end -->

## Go deeper

- [Search Platform Scaling](<Search_Platform_Scaling.md>)
- [Search Relevance](<Search_Relevance.md>)

## Implementations (tools)

- [Elasticsearch](<../../../../TOOLS/Elasticsearch/_index.md>)

## Practice questions

- [Intermediate] [Explain the difference between keyword and text fields and the multi-field pattern. Give a case where sorting or aggregating breaks because someone only mapped a field as text.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24447_Explain_the_difference_between_keyword_and_text_fields_and_t.md>)
- [Deep] [Your p99 search latency spiked from 40ms to 900ms with no traffic change. Walk me through the systematic investigation using the slow log, hot threads, and node stats to find the culprit.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24359_Your_p99_search_latency_spiked_from_40ms_to_900ms_with_no_tr.md>)
- [Deep] [Explain how the query cache and the request cache differ, what makes a query cacheable, and why using now in a range filter silently destroys cache hit rate.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24468_Explain_how_the_query_cache_and_the_request_cache_differ_wha.md>)
- [Deep] [Why should JVM heap stay at or below ~50% of RAM and under ~30 GB? Explain compressed oops and how the OS page cache serves Lucene, and what happens when you over-allocate heap.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24486_Why_should_JVM_heap_stay_at_or_below_50_of_RAM_and_under_30_.md>)
- [Deep] [Explain the search and write thread pools and their bounded queues. When you see EsRejectedExecutionException, what does it actually mean and how do you respond correctly rather than just raising the queue size?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24505_Explain_the_search_and_write_thread_pools_and_their_bounded_.md>)
- [Deep] [Design an alias strategy so your application never talks to a physical index directly. How does this enable blue-green reindexing, and what's the atomicity guarantee of alias swaps?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24524_Design_an_alias_strategy_so_your_application_never_talks_to_.md>)
- [Deep] [How would you design search over a rapidly changing inventory where stock counts update every second? Discuss update volume, refresh cost, and separating volatile fields from stable ones.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24542_How_would_you_design_search_over_a_rapidly_changing_inventor.md>)
- [Expert] [How would you integrate a learning-to-rank model into Elasticsearch? Explain feature logging, offline training, and serving the model at query time without wrecking p99.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24499_How_would_you_integrate_a_learning-to-rank_model_into_Elasti.md>)

## Related concepts

- [Inverted Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>) *(Data Structures and Algorithms)*
- [Re-ranking](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Re-ranking.md>) *(Retrieval and RAG)*
- [Index Strategy and Governance at Scale](<../../../Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Index Tuning in Practice](<../../../Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*
- [Sharding Vector Indexes](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Sharding_Vector_Indexes.md>) *(Distributed Systems Core)*
- [Distributed Joins in MPP Systems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>) *(Distributed Systems Core)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Security and Governance › Amazon OpenSearch Service
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › The Dangers of Defaults

