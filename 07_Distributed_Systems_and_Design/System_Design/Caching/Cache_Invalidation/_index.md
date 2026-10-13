---
id: "07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Caching"
tools: ["Redis"]
---

# Cache Invalidation

**L4 Advanced** · [↑ Caching](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Cache Failover and Resilience](<Cache_Failover_and_Resilience.md>)
- [Caching and Service Independence](<Caching_and_Service_Independence.md>)

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Your cache and DB disagree after a deploy. Describe your step-by-step approach to diagnose the inconsistency.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05325_Your_cache_and_DB_disagree_after_a_deploy_Describe_your_step.md>)
- [Intermediate] [You are asked to remove a caching layer that a team added prematurely. How do you prove it is safe to remove, ramp down traffic, and confirm the DB can absorb the load?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05598_You_are_asked_to_remove_a_caching_layer_that_a_team_added_pr.md>)
- [Deep] [What is cache invalidation and why is it considered a hard problem?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05518_What_is_cache_invalidation_and_why_is_it_considered_a_hard_p.md>)
- [Deep] [Your cache invalidation is done via async events and occasionally an event is dropped, leaving a stale entry forever. How do you make invalidation self-healing?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05426_Your_cache_invalidation_is_done_via_async_events_and_occasio.md>)
- [Deep] [Your service caches a value derived from a join across five tables and you cannot cheaply know which rows changed. How would you design invalidation that trades some over-invalidation for guaranteed correctness, and how do you tune that trade-off?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05525_Your_service_caches_a_value_derived_from_a_join_across_five_.md>)
- [Deep] [How would you design a cache so that a schema migration renaming a column can proceed without every cached row becoming instantly invalid and stampeding the freshly-migrated DB?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05566_How_would_you_design_a_cache_so_that_a_schema_migration_rena.md>)
- [Deep] [You cache derived data that depends on a config value. When the config changes, thousands of derived entries must recompute. How do you spread that recompute over time to protect the origin?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05613_You_cache_derived_data_that_depends_on_a_config_value_When_t.md>)
- [Expert] [You cache a value that is derived from three upstream services. When one upstream changes, how do you know which cached entries to invalidate without over-invalidating?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05423_You_cache_a_value_that_is_derived_from_three_upstream_servic.md>)

## Related concepts

- [Read-through and Write-behind Caching](<../../System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>) *(System Design Core)*
- [Cache Invalidation Across Regions](<../../../Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>) *(Distributed Systems Core)*
- [Distributed Cache Design](<../../High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*

