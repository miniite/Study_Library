---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: ["Redis", "Apache_Kafka"]
---

# Clickstream Counting Pipeline

**L4 Advanced** · [↑ System Design Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Primary-replica Databases](<Primary-replica_Databases.md>)

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [Design a simple leaderboard for a game showing top scores.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42243_Design_a_simple_leaderboard_for_a_game_showing_top_scores.md>)
- [Intermediate] [Design a distributed unique-counter for real-time 'X people viewing this product' that's cheap and roughly accurate. Why not a per-request DB increment, and what's your decay strategy?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42435_Design_a_distributed_unique-counter_for_real-time_X_people_v.md>)
- [Deep] [How would you count unique visitors to a website approximately?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42245_How_would_you_count_unique_visitors_to_a_website_approximate.md>)
- [Deep] [How would you compute trending items over a sliding time window efficiently?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42375_How_would_you_compute_trending_items_over_a_sliding_time_win.md>)
- [Deep] [Design a globally-unique, roughly time-ordered ID scheme without a central counter. Compare Snowflake IDs, ULIDs, and UUIDv7 on index locality, clock-skew risk, and coordination cost.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42172_Design_a_globally-unique_roughly_time-ordered_ID_scheme_with.md>)
- [Deep] [How would you design storage and retrieval for time-series metrics (billions of points/day) so recent data is fast and old data is cheap? Discuss downsampling, retention, and rollups.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42449_How_would_you_design_storage_and_retrieval_for_time-series_m.md>)
- [Deep] [Design a soft-delete + archival strategy for a table that grows 50M rows/month. Compare partitioning by time, a separate cold store, and TTL, on query cost, storage cost, and GDPR deletion.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42503_Design_a_soft-delete_archival_strategy_for_a_table_that_grow.md>)
- [Deep] [Design storage for time-series metrics at billions of points/day so recent data is fast and old data is cheap. Discuss downsampling, rollups, retention tiers, and why a columnar layout helps range queries.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42536_Design_storage_for_time-series_metrics_at_billions_of_points.md>)

## Related concepts

- [Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*
- [Search Backend Design](<../../High_Level_Design/Search_Backend_Design.md>) *(High Level Design)*
- [Binary Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>) *(Data Structures and Algorithms)*

