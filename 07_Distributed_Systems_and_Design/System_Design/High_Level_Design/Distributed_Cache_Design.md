---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["AWS", "Apache_Kafka", "Redis"]
---

# Distributed Cache Design

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Design a resumable file-upload frontend for large videos on flaky mobile networks — chunking, retry, progress, and pause/resume across app restarts; where do you keep upload state?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29627_Design_a_resumable_file-upload_frontend_for_large_videos_on_.md>)
- [Deep] [Design a distributed cache layer in front of a database — how do you choose write-through vs write-back vs cache-aside, and how do you handle stampede on a hot key expiry?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29640_Design_a_distributed_cache_layer_in_front_of_a_database_how_.md>)
- [Deep] [Design a distributed cache cluster (Memcached/Redis-style) that must scale from 10 to 200 nodes — how do you use consistent hashing with virtual nodes to minimize key movement during scale-out?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29705_Design_a_distributed_cache_cluster_Memcached_Redis-style_tha.md>)
- [Deep] [Design the sharding strategy for a multi-tenant OLTP database where tenant sizes vary 1000x — how do you place small tenants together isolate whales and rebalance when a tenant outgrows its shard?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29741_Design_the_sharding_strategy_for_a_multi-tenant_OLTP_databas.md>)
- [Deep] [Design a CDN's origin-shield and cache-fill layer so a cache miss on a hot object doesn't stampede the origin — how do you coalesce fills tier the cache and handle cache-key normalization?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29768_Design_a_CDN_s_origin-shield_and_cache-fill_layer_so_a_cache.md>)
- [Expert] [A URL shortener uses a distributed counter for key generation — how do you shard the counter across regions without a single point of contention, and how do you avoid gaps or reuse?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29584_A_URL_shortener_uses_a_distributed_counter_for_key_generatio.md>)
- [Expert] [Design exactly-once semantics for a message pipeline that consumes from Kafka and writes to Postgres — how do you use transactional offsets or the transactional-outbox to avoid duplicates and lost messages?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29713_Design_exactly-once_semantics_for_a_message_pipeline_that_co.md>)
- [Expert] [Design a zero-downtime migration off a proprietary vendor database to an open engine at petabyte scale — how do you sequence dual-writes, parity verification, and the point of no return?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29797_Design_a_zero-downtime_migration_off_a_proprietary_vendor_da.md>)

## Related concepts

- [Read Your Writes with Replicas](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*
- [Read-through and Write-behind Caching](<../System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>) *(System Design Core)*
- [Two-level Cache Design](<../Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Friends Don't Let Friends Do Dual-Writes

