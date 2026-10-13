---
id: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Maintaining_Denormalized_Views"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design"
tools: []
---

# Maintaining Denormalized Views

**L5 Expert** · [↑ Low Level Design](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Design a versioned document/wiki with revision history. How do you model a diff and rollback?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31569_Design_a_versioned_document_wiki_with_revision_history_How_d.md>)
- [Intermediate] [Design a soft-delete + restore model across an aggregate with child rows; how do you keep queries clean without every reader remembering the deleted filter, and how do foreign-key constraints and unique indexes behave?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31691_Design_a_soft-delete_restore_model_across_an_aggregate_with_.md>)
- [Deep] [Design a moving-average or sliding-window rate calculator class. How do you evict old data points?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31560_Design_a_moving-average_or_sliding-window_rate_calculator_cl.md>)
- [Deep] [Your infinite-scroll feed re-renders the whole list on every append and janks on low-end phones; how do you redesign the component's data structure and keys to keep appends O(1) and avoid re-render storms?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31646_Your_infinite-scroll_feed_re-renders_the_whole_list_on_every.md>)
- [Deep] [Your Q&A reputation computation is recalculated on every page load and is now the slowest query; how do you redesign so reputation is maintained incrementally and stays consistent after a vote is retracted?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31658_Your_Q_A_reputation_computation_is_recalculated_on_every_pag.md>)
- [Deep] [Your read replica lags and users hit read-your-writes bugs after a POST; design a class-level 'sticky-to-primary' or version-token approach so a user reads their own write consistently without forcing all reads to primary.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31715_Your_read_replica_lags_and_users_hit_read-your-writes_bugs_a.md>)
- [Deep] [You must decide how the org models 'derived vs stored' fields (an order total: recompute on read or store and keep in sync); walk through the consistency, performance, and audit trade-offs and set a guideline for when each is correct at scale.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31920_You_must_decide_how_the_org_models_derived_vs_stored_fields_.md>)
- [Deep] [You're deciding how the org models 'a computed/denormalized read model for the frontend' (a dashboard aggregate) so the backend owns the shape; how do you version the payload so an old app client tolerates a new field being added or an optional field going missing?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31941_You_re_deciding_how_the_org_models_a_computed_denormalized_r.md>)

## Related concepts

- [Replica Staleness](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness.md>) *(SQL Fundamentals)*
- [Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Denormalization Trade-offs and Patterns](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>) *(Normalization)*

