---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: ["Redis"]
---

# Read-through and Write-behind Caching

**L4 Advanced** · [↑ System Design Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Frontend Graceful Degradation](<Frontend_Graceful_Degradation.md>)
- [Multi-tenant Migrations](<Multi-tenant_Migrations.md>)
- [Coordinated Client Upgrades](<Coordinated_Client_Upgrades.md>)
- [Org-wide Lineage and PII Tracking](<Org-wide_Lineage_and_PII_Tracking.md>)

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Explain common cache eviction policies like LRU and LFU.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42221_Explain_common_cache_eviction_policies_like_LRU_and_LFU.md>)
- [Intermediate] [Design memory eviction for a Redis cache holding both must-keep session data and evictable cache entries in the same instance. How do you stop maxmemory eviction from dropping sessions?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42498_Design_memory_eviction_for_a_Redis_cache_holding_both_must-k.md>)
- [Deep] [How would you check permissions on every request without slowing down the API?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42380_How_would_you_check_permissions_on_every_request_without_slo.md>)
- [Deep] [Design the storage and dedupe for a Dropbox-like sync where identical file blocks across users are stored once. How does content-addressed chunking work and where does it break down?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42454_Design_the_storage_and_dedupe_for_a_Dropbox-like_sync_where_.md>)
- [Deep] [Explain how MVCC lets readers avoid blocking writers in Postgres and what causes table bloat. How would you tune autovacuum for a high-churn table?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42494_Explain_how_MVCC_lets_readers_avoid_blocking_writers_in_Post.md>)
- [Deep] [Explain HTTP caching end-to-end: ETag vs Last-Modified, Cache-Control max-age vs s-maxage, and stale-while-revalidate. How would you cache a personalized API response safely at a CDN?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42520_Explain_HTTP_caching_end-to-end_ETag_vs_Last-Modified_Cache-.md>)
- [Deep] [Design read-through + write-behind caching for user profiles. What's your invalidation approach, how do you bound data loss if the cache dies before the write-behind flush, and when is write-behind too risky?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42567_Design_read-through_write-behind_caching_for_user_profiles_W.md>)
- [Expert] [What is a cache stampede and how would you prevent it?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42281_What_is_a_cache_stampede_and_how_would_you_prevent_it.md>)

## Related concepts

- [Distributed Cache Design](<../../High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*
- [Caching and Service Independence](<../../Caching/Cache_Invalidation/Caching_and_Service_Independence.md>) *(Caching)*
- [Two-level Cache Design](<../../Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*

