---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Sharding_Vector_Indexes"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Sharding Vector Indexes

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [You store per-user long-term memory for an agent in a sharded KV store keyed by user id, and a few power users create hot shards. Explain how you'd detect the hot key and three mitigations, and how memory-write ordering matters for correctness.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23149_You_store_per-user_long-term_memory_for_an_agent_in_a_sharde.md>)
- [Deep] [You serve a 200GB user-embedding table that won't fit on one node for a recommender. Compare sharding by key, a parameter-server design, and a distributed KV store on latency and hot-key risk, and which you'd pick.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23194_You_serve_a_200GB_user-embedding_table_that_won_t_fit_on_one.md>)
- [Deep] [A distributed query fans out to 100 shards and its p99.9 is dominated by the single slowest shard. Explain tail-latency amplification for scatter-gather queries and how hedged/backup requests to replicas mitigate it, plus the extra load they create.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24138_A_distributed_query_fans_out_to_100_shards_and_its_p99_9_is_.md>)
- [Deep] [One user embedding (a celebrity) is looked up 100x more than others in your recommender's distributed embedding store, creating a hot shard. How would you detect and mitigate this hot key at serving time?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23634_One_user_embedding_a_celebrity_is_looked_up_100x_more_than_o.md>)
- [Deep] [You're designing consistent-hashing-based data placement for a distributed store and want to bound the load any single node can take. Explain why plain consistent hashing with vnodes is insufficient under a hot key, and how bounded-load consistent hashing or explicit hot-key sharding fixes it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24182_You_re_designing_consistent-hashing-based_data_placement_for.md>)
- [Deep] [A database read that fans out to 100 shards has p99.9 dominated by the slowest shard. Explain the slowest-of-N statistics, and design hedged/tied reads at the data layer, quantifying where the extra query load makes hedging counterproductive.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24239_A_database_read_that_fans_out_to_100_shards_has_p99_9_domina.md>)
- [Expert] [You're designing shuffle-sharding as the org's default multi-tenant isolation. Derive why shuffle sharding reduces the probability that any two tenants share a full failure domain, and how you'd size virtual shards for 50k tenants so a poison tenant can't blast more than a bounded fraction.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23537_You_re_designing_shuffle-sharding_as_the_org_s_default_multi.md>)
- [Expert] [Explain how you'd design shuffle-sharding for a multi-tenant AIOps SaaS so any single abusive tenant's telemetry flood can only affect a bounded, non-overlapping subset of ingestion/query capacity. Derive why shuffle sharding beats simple sharding for isolating a poison tenant across thousands of tenants.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24039_Explain_how_you_d_design_shuffle-sharding_for_a_multi-tenant.md>)

## Related concepts

- [Distributed Cache Design](<../../../System_Design/High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*
- [Clickstream Counting Pipeline](<../../../System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*
- [Search Backend Design](<../../../System_Design/High_Level_Design/Search_Backend_Design.md>) *(High Level Design)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — other very much. For example, at eBay, the users are in one shard, and the items for sale are in
- *Ultimate AWS Data Engineering* — Rathish Mohan — Partitioning and Sharding Optimizing Data Management › Introduction
- *Ultimate AWS Data Engineering* — Rathish Mohan — Partitioning and Sharding Optimizing Data Management › Sharding in Distributed Systems
- *Ultimate AWS Data Engineering* — Rathish Mohan — Partitioning and Sharding Optimizing Data Management › Implementing Sharding in Cloud-Based Distributed Systems
- *Ultimate AWS Data Engineering* — Rathish Mohan — Partitioning and Sharding Optimizing Data Management › Sharding Strategies
- *Think Distributed Systems* — Dominik Tornow — Partitioning › 7.7 Consistent hashing
- *Ultimate Data Engineering with Databricks* — Mayank Malhotra — Chapter 8: Performance Tuning and Optimization › Analyzing Query Execution Plans and Identifying Performance Hotspots

