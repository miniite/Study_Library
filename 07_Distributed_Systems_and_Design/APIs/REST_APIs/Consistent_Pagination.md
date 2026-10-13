---
id: "07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/REST_APIs"
tools: ["Python"]
---

# Consistent Pagination

**L4 Advanced** · [↑ REST APIs](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Deep] [What is the difference between offset-based and cursor-based pagination? When would you prefer cursor-based?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38339_What_is_the_difference_between_offset-based_and_cursor-based.md>)
- [Deep] [A junior engineer's PR adds GET /orders/search that runs an unbounded query. What would you flag in review about pagination, injection, timeouts, indexing, and abuse, and how do you teach the pattern?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38599_A_junior_engineer_s_PR_adds_GET_orders_search_that_runs_an_u.md>)
- [Deep] [A REST endpoint aggregates data from Postgres and Elasticsearch and the two drift apart during reindexing, so callers see inconsistent counts. How would you make the endpoint present a coherent view during reindex?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38646_A_REST_endpoint_aggregates_data_from_Postgres_and_Elasticsea.md>)
- [Deep] [Your API must return a consistent ordering for a leaderboard endpoint under heavy concurrent score updates. How do you design pagination and tie-breaking so a client scrolling doesn't see items jump around?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38666_Your_API_must_return_a_consistent_ordering_for_a_leaderboard.md>)
- [Deep] [A REST endpoint occasionally returns duplicate items in a paginated list under concurrent writes because it sorts by a mutable field. How do you diagnose this and design a stable sort key that survives updates?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38738_A_REST_endpoint_occasionally_returns_duplicate_items_in_a_pa.md>)
- [Expert] [How would you design pagination that stays consistent when new records are being inserted between page requests?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38509_How_would_you_design_pagination_that_stays_consistent_when_n.md>)
- [Expert] [An endpoint's read-after-write consistency broke when you added a read replica: clients POST then immediately GET stale data. How do you diagnose and fix this while keeping reads on replicas?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38597_An_endpoint_s_read-after-write_consistency_broke_when_you_ad.md>)
- [Expert] [You expose GET /accounts/:id/balance and it must reflect writes made microseconds earlier from the same client but tolerate replica lag for everyone else. How do you route reads to satisfy both without a strong-read for all?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38645_You_expose_GET_accounts_id_balance_and_it_must_reflect_write.md>)

## Related concepts

- [Exactly Once Dedup Structures](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>) *(Data Structures and Algorithms)*
- [Index Strategy and Governance at Scale](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Maintaining Denormalized Views](<../../System_Design/Low_Level_Design/Idempotent_Consumer_Design/Maintaining_Denormalized_Views.md>) *(Low Level Design)*

