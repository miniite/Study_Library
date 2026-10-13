---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Search_Backend_Design"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["Redis"]
---

# Search Backend Design

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Design a faceted-search/aggregation service (filters by brand price color) over a large index — how do you compute facet counts efficiently and keep them consistent with the filtered result set?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29722_Design_a_faceted-search_aggregation_service_filters_by_brand.md>)
- [Deep] [Design a ride-sharing backend's matching service that pairs riders to nearby drivers in real time — how do you index and query driver locations geospatially at city scale?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29615_Design_a_ride-sharing_backend_s_matching_service_that_pairs_.md>)
- [Deep] [Design a URL/web crawler that fetches billions of pages — how do you handle politeness, dedup of URLs, distributed frontier management, and re-crawl scheduling?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29624_Design_a_URL_web_crawler_that_fetches_billions_of_pages_how_.md>)
- [Deep] [Explain how you'd data-model a many-to-many social graph (follows/friends) in a wide-column store to support both 'who do I follow' and 'who follows me' efficiently.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29651_Explain_how_you_d_data-model_a_many-to-many_social_graph_fol.md>)
- [Deep] [Design a recommendation feed's serving layer (not the ML) — how do you fetch candidates, apply real-time filters, rank, and cache to hit a 100ms budget?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29678_Design_a_recommendation_feed_s_serving_layer_not_the_ML_how_.md>)
- [Deep] [Search p99 spiked after a big reindex while p50 stayed flat — walk through how you'd diagnose whether it's segment merges GC cold caches or a few heavy queries and how you'd protect tail latency.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29723_Search_p99_spiked_after_a_big_reindex_while_p50_stayed_flat_.md>)
- [Deep] [Design a geospatial proximity backend for finding nearby drivers that ingests millions of location updates/sec — compare geohash-bucketed indexing vs an in-memory grid on update cost and query latency.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29759_Design_a_geospatial_proximity_backend_for_finding_nearby_dri.md>)
- [Deep] [Design a matchmaking service that pairs players by skill latency and queue time at scale — how do you balance match quality against wait time shard the pools by region and avoid starving edge-skill players?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29956_Design_a_matchmaking_service_that_pairs_players_by_skill_lat.md>)

## Related concepts

- [Clickstream Counting Pipeline](<../System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*
- [Index Strategy and Governance at Scale](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Stream Join Data Structures](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*

