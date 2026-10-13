---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Rollups_vs_On-demand_Queries"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS"]
---

# Rollups vs On-demand Queries

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Basic] [What is a materialized view in the warehouse and how could you use one to serve a stable, cheap feature to your model?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10699_What_is_a_materialized_view_in_the_warehouse_and_how_could_y.md>)
- [Deep] [Explain how you'd store deduplicated 'latest state per entity' derived from an append-only event log so a service can look it up quickly. What are your serving-layer options?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10542_Explain_how_you_d_store_deduplicated_latest_state_per_entity.md>)
- [Deep] [Your warehouse queries for training hit a concurrency limit during a big experiment sweep. How do you isolate ML workloads from analytics workloads?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10732_Your_warehouse_queries_for_training_hit_a_concurrency_limit_.md>)
- [Deep] [You need to prove EU customer rows physically never leave EU storage even though a global query engine reads them. Where might a shuffle or result cache accidentally move data across borders, and how do you prevent it?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11132_You_need_to_prove_EU_customer_rows_physically_never_leave_EU.md>)
- [Deep] [How do you decide between materialized pre-aggregated rollups versus on-demand query for a high-cardinality observability workload (per-endpoint per-region latency)? Map the storage/compute/freshness triangle and how it shifts at 100x cardinality for AIOps.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10935_How_do_you_decide_between_materialized_pre-aggregated_rollup.md>)
- [Deep] [How do you architect cost and performance isolation between ad-hoc analyst queries and scheduled production pipelines sharing one warehouse, when a runaway analyst query can starve a revenue-critical pipeline? Compare separate virtual warehouses, workload management queues, and query priority, and where each fails.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10999_How_do_you_architect_cost_and_performance_isolation_between_.md>)
- [Expert] [Design the partitioning and clustering strategy for a multi-petabyte feature fact table with heterogeneous, evolving training access patterns. What's the cost of getting it wrong, and how do you re-partition live without stalling model pipelines?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10813_Design_the_partitioning_and_clustering_strategy_for_a_multi-.md>)
- [Expert] [A principal peer proposes deleting the specialized TSDB and OLAP stores and running all AIOps queries directly on the lake with Trino. Stress-test that: where does 'just query the lake' fall over for alerting query concurrency, freshness, and sub-second dashboards?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10919_A_principal_peer_proposes_deleting_the_specialized_TSDB_and_.md>)

## Related concepts

- [Choosing a Database for the Workload](<../../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Data Warehouse Fundamentals](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Scalability and Performance Optimization

