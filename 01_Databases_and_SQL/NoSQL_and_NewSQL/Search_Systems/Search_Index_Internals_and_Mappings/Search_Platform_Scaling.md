---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Platform_Scaling"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems"
tools: ["Elasticsearch"]
---

# Search Platform Scaling

**L5 Expert** · [↑ Search Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Scaling a search platform is mostly about shard count and size, tiered storage and lifecycle, tenant isolation, query guard rails and recoverability — a search cluster is a derived, rebuildable index, so design it to be capped, observable and replaceable.

### 1. The big picture
A search cluster (Elasticsearch/OpenSearch) scales by adding shards and nodes, but each shard has a fixed overhead and every index, field and shard lives in *cluster state* handled by the elected master. Growth problems therefore show up as: too many shards (master and heap pressure), too few large shards (slow recovery, no parallelism), noisy tenants (one expensive aggregation starving everyone), runaway mappings, and no cost attribution. At platform scale you also need contracts (SLOs, quotas), disaster recovery, and a policy for how tenants map to indexes.

Because search is normally a *derived* store — the system of record lives elsewhere — the DR posture is different from a database: rebuild-from-source is often better than restoring a snapshot.

### 2. Core concepts and vocabulary
- **Primary shard / replica** — write partition and its copies; primaries are fixed at index creation.
- **Cluster state** — metadata (indices, mappings, shard routing) replicated to all nodes; changes pass through the master.
- **Node roles** — master-eligible, data (hot/warm/cold/frozen), ingest, coordinating-only, ML.
- **ILM / ISM** — index lifecycle management (Elasticsearch ILM, OpenSearch ISM): rollover → warm → cold → frozen → delete.
- **Rollover / data stream** — write alias or stream that creates a new backing index when size, age or doc count is hit.
- **Circuit breaker** — memory guard that rejects a request instead of letting the node OOM.
- **Search backpressure / thread-pool queue** — limits on concurrent work; rejections surface as 429s.
- **Index-per-tenant vs shared index with tenant filter (filtered alias, routing)** — the two multi-tenancy primitives.
- **Early vs late binding ACLs** — baking permissions into the index vs filtering at query time.
- **CCS / CCR** — cross-cluster search and cross-cluster replication.

### 3. How it works, step by step — a scaling playbook
1. **Size shards.** Aim for roughly 10–50 GB per shard (smaller for search-heavy, bigger for logs), and plan total shard count against heap: a common rule of thumb is no more than ~20 shards per GB of heap; recent versions also cap shards per node (`cluster.max_shards_per_node`, default 1000).
2. **Split the data by lifecycle.** Time-based data → data streams with ILM; hot nodes get fast SSD, warm/cold cheaper storage, frozen uses searchable snapshots on object storage.
3. **Pick a tenancy model.** Small tenants share indexes (with a `tenant_id` filter and custom routing); big tenants get dedicated indexes or clusters. Automate promotion from shared to dedicated at thresholds.
4. **Constrain queries.** Set `index.max_result_window` (default 10,000), `search.max_buckets`, query timeouts, `allow_expensive_queries: false` when appropriate, restrict scripts and wildcard patterns; page with `search_after` + point-in-time.
5. **Separate roles.** Dedicated masters (3), separate coordinating nodes for heavy aggregations, ingest nodes for pipelines.
6. **Isolate failure domains.** Zone/rack awareness (`cluster.routing.allocation.awareness.attributes`), replicas ≥ 1, clusters per domain for blast-radius isolation.
7. **Plan recovery.** Snapshots to object storage on a schedule, tested restores, and a documented rebuild path from the source of truth.

### 4. Worked example
Time-based log index with ILM and a tenant-aware template:
```json
PUT _index_template/logs
{
  "index_patterns": ["logs-*"],
  "data_stream": {},
  "template": {
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1,
      "index.lifecycle.name": "logs-policy",
      "index.mapping.total_fields.limit": 500
    },
    "mappings": { "dynamic": "false",
      "properties": { "@timestamp": {"type":"date"}, "tenant": {"type":"keyword"},
                      "message": {"type":"text"}, "attrs": {"type":"flattened"} } }
  }
}

PUT _ilm/policy/logs-policy
{ "policy": { "phases": {
  "hot":    { "actions": { "rollover": { "max_primary_shard_size": "40gb", "max_age": "1d" } } },
  "warm":   { "min_age": "7d",  "actions": { "shrink": {"number_of_shards": 1}, "forcemerge": {"max_num_segments": 1} } },
  "cold":   { "min_age": "30d", "actions": { "searchable_snapshot": { "snapshot_repository": "s3-repo" } } },
  "delete": { "min_age": "90d", "actions": { "delete": {} } } } } }
```
Check health with `GET _cat/shards?v&s=store:desc`, `GET _cluster/allocation/explain`, `GET _cluster/stats`, and `GET _nodes/stats/breaker,thread_pool`. A node with many `search` rejections and high `fielddata` or request breaker trips indicates a query problem, not a capacity problem.

### 5. Under the hood
- **Why too many shards hurts.** Each shard is a Lucene index with its own segments, file handles, caches and heap structures; every change to the shard map is a cluster-state update. With thousands of shards, cluster-state publishing slows, master heap grows, recoveries serialize, and a single node loss triggers a storm of relocations. Consolidation: reindex into fewer shards, shrink API, or merge time-based indexes by period.
- **Why too few/large hurts.** Recovery after node failure copies the whole shard; a 500 GB shard takes hours, and one query can't parallelise beyond shard count.
- **Query fan-out.** A search hits one copy of *every* shard of the targeted indexes; latency = slowest shard. Adding replicas raises throughput (more copies to choose from, with adaptive replica selection) but not single-query speed.
- **Deep paging** costs `from + size` per shard in memory, hence the 10,000 limit. `search_after` with a sort key (and a PIT for a consistent view) is stateless and cheap; `scroll` suits exports.
- **Mapping explosion** is a cluster-state attack: tenants adding unique field names. Guard rails: `dynamic: false/strict`, field-count limits, `flattened`, per-tenant onboarding with schema review, and monitoring of mapping size per index.
- **Permission-aware search.** *Early binding* stores ACL tokens on each document and filters on them (fast query, expensive permission changes, reindex on ACL change). *Late binding* checks permissions after retrieval (always current, but top-N may be filled with documents the user can't see, causing paging gaps). Hybrid: coarse early filter + late verify.
- **Derived-data DR.** If rebuild is hours and source is reliable, rebuild may beat snapshot restore; snapshots shine when the reindex is days long or the source is expensive to replay. A corrupt shard replicates to replicas and into later snapshots, so keep older snapshot generations and checksum/restore-test.

### 6. Performance and design implications
- Track the **golden signals**: query latency p50/p99, indexing rate, thread-pool rejections, JVM heap and GC time, segment count and merge time, disk watermarks (85/90/95%), shard count, cluster-state size and publish time.
- **Hot-warm** cuts cost: SSD only for the data that's queried. Frozen/searchable snapshots can give ~10× storage savings at higher latency.
- **Bulk sizing** 5–15 MB, parallel workers, `refresh_interval` 30 s; avoid per-document updates of hot docs (each is delete+insert).
- **Cost attribution** requires tags at the index/alias level (tenant → index naming), query tagging via `X-Opaque-Id`, and per-index storage stats; chargeback uses storage + indexing + search CPU shares.
- **SLO tiers**: platinum tenants get dedicated hot nodes and tighter query limits; best-effort tenants share. Publish quotas and rate limits, and allow temporary burst credits.
- **Change safety**: roll out mappings/analyzers via new index + alias swap; propagate configuration via staged rollouts (canary → region → global) to avoid global config-push outages.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `_cluster/allocation/explain` | Why a shard is unassigned | Red/yellow cluster |
| `_shrink`, `_split`, `_reindex` | Change shard count | Consolidate or grow |
| `cluster.max_shards_per_node` | Shard cap guard | Prevent runaway counts |
| `index.max_result_window`, `search_after`, PIT | Deep paging control | APIs with pagination |
| `search.default_search_timeout`, `terminate_after` | Bound query cost | Multi-tenant platforms |
| ILM/ISM policies | Lifecycle automation | Time series |
| `_snapshot` + SLM | Backup scheduling | DR |
| `X-Opaque-Id` header + slowlog | Attribute slow queries | Cost/ownership |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| One shared big cluster | Cost efficiency matters and tenants are similarly sized and trusted |
| Cluster per domain | Blast radius, compliance, or different SLAs matter more than utilisation |
| Index per tenant | Few large tenants; per-tenant mappings/retention needed |
| Shared index + tenant filter/routing | Thousands of small tenants — avoids shard explosion |
| Searchable snapshots | Latency-insensitive historical queries |
| Snapshot restore for DR | Rebuild from source is slower than restore |

### 9. Common mistakes
- Shard-per-day indexes of a few MB → thousands of shards → heap and master stress → monthly/size-based rollover.
- Letting any client run any aggregation → one query takes the cluster → limits, timeouts, circuit breakers, query review.
- Global config pushed at once → cascading multi-region failure → staged rollouts, blast-radius limits.
- No cost attribution → can't charge back or find offenders → naming conventions, tags, slow-log IDs.
- Snapshot never test-restored → useless in an incident → scheduled restore drills.
- One huge shared index for 5-million-doc and 5-doc tenants → hot tenant starves others → size-based promotion to dedicated index.

### 10. 🎯 Interview kit
**30-second answer:** Keep shards 10–50 GB and few, use ILM to tier data, separate master/data/coordinating roles, cap query cost, pick tenancy by tenant size, and treat the cluster as a rebuildable derived store with tested snapshots.

**Structure a longer answer:** workload & SLOs → shard strategy → lifecycle/tiering → tenancy → guard rails → observability/cost → DR.

**Likely follow-ups:**
- *Too many shards: effect and plan?* → master/heap/cluster-state; shrink or reindex into rollover-by-size, no downtime via alias swap.
- *Shared cluster vs per-domain?* → utilisation vs blast radius; mix based on tier.
- *ACL design?* → early vs late binding trade-offs.
- *Rebuild vs restore?* → depends on reindex duration and source reliability.

**Signals interviewers look for:** capacity numbers, awareness of cluster-state limits, platform thinking (quotas, SLO tiers, cost attribution), DR for derived data.

### 11. Check yourself
- [ ] Calculate shard count and heap for 20 TB hot data with 30 GB shards.
- [ ] Draw an ILM policy for 90-day retention with searchable snapshots.
- [ ] Explain why `from=100000,size=10` is bad and the alternative.
- [ ] Design the switch from shared index to dedicated index for a growing tenant.
- [ ] Describe a staged config rollout for a global search platform.
<!-- explainer:end -->

## Implementations (tools)

- [Elasticsearch](<../../../../TOOLS/Elasticsearch/_index.md>)

## Practice questions

- [Intermediate] [You're designing the mapping for a product catalog with 50 fields but only 8 are ever searched. How do you keep the mapping lean, and what's the cost of leaving everything as default dynamic mapping?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24492_You_re_designing_the_mapping_for_a_product_catalog_with_50_f.md>)
- [Deep] [Design the observability and SLO stack for search as a platform so tenants can self-diagnose their own relevance and latency without paging your team. What metrics do you expose per-tenant, and how do you attribute cluster-level pain to specific tenants?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24389_Design_the_observability_and_SLO_stack_for_search_as_a_platf.md>)
- [Deep] [An architecture review board asks you to justify why search is a platform and not just a library each team imports. Make the crisp argument for the platform boundary, what belongs in the platform versus a shared library, and where you've seen the boundary drawn wrong.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24437_An_architecture_review_board_asks_you_to_justify_why_search_.md>)
- [Expert] [You've been handed a search bill of 3 million dollars a year across a hundred clusters with 30 percent average utilization. Lay out a FinOps program to cut it 40 percent in a year without regressing p99 or reliability, and how you'd defend the plan to finance.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24355_You_ve_been_handed_a_search_bill_of_3_million_dollars_a_year.md>)
- [Expert] [You must set data-governance and compliance architecture for a search platform holding PII across GDPR, CCPA, and India's DPDP jurisdictions. How do you architect right-to-be-forgotten, data residency, and field-level access without crippling query performance?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24372_You_must_set_data-governance_and_compliance_architecture_for.md>)
- [Expert] [Your search platform's indexing pipeline is the bottleneck at 2M docs/sec across tenants. Redesign the write path — backpressure, prioritization, fair-sharing across tenants — so a bulk backfill from one tenant can't starve real-time updates for others.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24395_Your_search_platform_s_indexing_pipeline_is_the_bottleneck_a.md>)
- [Expert] [You're deciding between Elasticsearch's cross-cluster replication and building your own Kafka-based dual-ingest for a globally-distributed search platform. Compare the operational ownership, consistency lag, and failure-recovery ergonomics, and which you'd bet on for a decade.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24417_You_re_deciding_between_Elasticsearch_s_cross-cluster_replic.md>)
- [Expert] [You need to architect blue-green for the entire search platform so you can roll a risky cluster-wide change (new ES major, new hardware) with instant rollback. What's the traffic-shifting, index-syncing, and cost model, and when is blue-green not worth the double footprint?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24438_You_need_to_architect_blue-green_for_the_entire_search_platf.md>)

## Related concepts

- [Index Strategy and Governance at Scale](<../../../Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Search Backend Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Search_Backend_Design.md>) *(High Level Design)*
- [Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>) *(Data Architecture)*
- [Open vs Proprietary Formats](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats.md>) *(Data Platform Strategy)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Resource Allocation (DTUs vs. vCores)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Real-World Use Cases
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Elastic Pools in Multi-Tenant SaaS Architectures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Elastic Jobs for Management at Scale
- *Data Engineering with Medallion Architecture* — Miki Eto — Medallion Architecture in Multi-cloud › Cautions and limits
- *The Enterprise Big Data Lake* — Alex Gorelik — Architecting the Data Lake › Cloud Data Lakes
- *The Enterprise Big Data Lake* — Alex Gorelik — Big Data in Medicine › E
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — E
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Security Design Patterns › Complexity
- *Data Engineering Design Patterns* — Bartosz Konieczny — K › M
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Challenge #1 -- platform architectures change rapidly › Platform architectures in the 21st century

