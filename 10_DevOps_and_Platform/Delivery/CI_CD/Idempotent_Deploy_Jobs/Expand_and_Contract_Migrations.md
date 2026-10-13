---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Expand_and_Contract_Migrations"
level: 5
type: "deep-dive"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Git_and_GitHub"]
---

# Expand and Contract Migrations

**L5 Expert** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)

## Practice questions

- [Intermediate] [Your alert-enrichment service pulls topology from a CMDB; how would you deploy a change to the enrichment logic without a stale CMDB snapshot in CI giving you false confidence?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06558_Your_alert-enrichment_service_pulls_topology_from_a_CMDB_how.md>)
- [Deep] [Your RAG deploy needs both a code change and a fresh index build; how do you order them so users never query new code against a stale index or old code against a new index?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06396_Your_RAG_deploy_needs_both_a_code_change_and_a_fresh_index_b.md>)
- [Deep] [You deploy a change to a dimension table's logic that requires re-processing history; how would you sequence the backfill and the code deploy so downstream marts never read a half-rebuilt table?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06678_You_deploy_a_change_to_a_dimension_table_s_logic_that_requir.md>)
- [Deep] [How would you deploy a change to a partitioned ingestion job so that a bad deploy only affects the current partition and prior partitions stay untouched and queryable, bounding the blast radius of a data bug?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06759_How_would_you_deploy_a_change_to_a_partitioned_ingestion_job.md>)
- [Expert] [How do you handle the case where re-embedding the whole corpus for an index migration takes six hours and can't block the deploy; walk me through dual-index, backfill, and read-switch phases.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06328_How_do_you_handle_the_case_where_re-embedding_the_whole_corp.md>)
- [Expert] [Why might a forward-only data transformation (e.g., an in-place UPDATE that overwrites source values) be un-rollbackable, and how would you design the deploy to keep a recovery path?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06744_Why_might_a_forward-only_data_transformation_e_g_an_in-place.md>)
- [Expert] [You need to deploy a schema change that renames a column used by three services; walk me through the full expand-and-contract sequence across code and DB so no service ever breaks.](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06928_You_need_to_deploy_a_schema_change_that_renames_a_column_use.md>)
- [Expert] [A deploy needs to create a Postgres index on a 300M-row table and a naive CREATE INDEX locked writes for 20 minutes; how would you run it concurrently and gate the app deploy on the index being ready without holding the pipeline hostage?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07410_A_deploy_needs_to_create_a_Postgres_index_on_a_300M-row_tabl.md>)

## Related concepts

- [Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*
- [Schema Migration Strategy](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>) *(Migrations)*
- [Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>) *(Cloud Fundamentals)*

