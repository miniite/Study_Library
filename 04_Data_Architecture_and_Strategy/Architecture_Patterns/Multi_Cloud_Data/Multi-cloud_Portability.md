---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Multi-cloud_Portability"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data"
tools: ["AWS", "Google_Cloud", "Azure", "Apache_Kafka"]
---

# Multi-cloud Portability

**L5 Expert** · [↑ Multi Cloud Data](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Google Cloud](<../../../TOOLS/Google_Cloud/_index.md>)
- [Azure](<../../../TOOLS/Azure/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [How would you debug a cross-cloud query that intermittently 40x-slows only when it reaches partitions stored in the other cloud's object store — what's your isolation methodology?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32911_How_would_you_debug_a_cross-cloud_query_that_intermittently_.md>)
- [Deep] [How do you keep Terraform/OpenTofu modules DRY across AWS/GCP/Azure when their primitives genuinely differ — where do you accept per-cloud modules versus forcing a leaky abstraction?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32923_How_do_you_keep_Terraform_OpenTofu_modules_DRY_across_AWS_GC.md>)
- [Deep] [Explain your strategy for testing a multi-cloud failover end to end without a real outage — how do you run game days that actually exercise the cross-cloud path?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32948_Explain_your_strategy_for_testing_a_multi-cloud_failover_end.md>)
- [Deep] [What's your read on serverless data services (BigQuery, Athena, Snowflake) for portability — do serverless abstractions make multi-cloud easier or lock you in harder, and why?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32974_What_s_your_read_on_serverless_data_services_BigQuery_Athena.md>)
- [Deep] [A team wants to run the same Spark job on EMR and Dataproc for portability — what subtle behavioral differences (shuffle, autoscaling, spot handling) will bite them, and how do you guard against it?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32988_A_team_wants_to_run_the_same_Spark_job_on_EMR_and_Dataproc_f.md>)
- [Expert] [Design a streaming backbone that spans two clouds with sub-second replication for a fraud-detection use case — what consistency guarantees do you sacrifice and how do you communicate that to risk teams?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32913_Design_a_streaming_backbone_that_spans_two_clouds_with_sub-s.md>)
- [Expert] [What is your stance on 'lift-and-shift to a second cloud' vs 're-architect for portability' during a mandated exit from a primary vendor — how do you sequence to limit business risk?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32939_What_is_your_stance_on_lift-and-shift_to_a_second_cloud_vs_r.md>)
- [Expert] [How do you handle the 'catalog of catalogs' problem — a global data-discovery experience over Unity Catalog + Glue + BigQuery + Purview — without building a fragile crawler that's always stale?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32949_How_do_you_handle_the_catalog_of_catalogs_problem_a_global_d.md>)

## Related concepts

- [Region Failure Resilience](<../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Region_Failure_Resilience.md>) *(Architecture Strategy)*
- [Multi-tenant Migrations](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*
- [Warehouse Vendor Exit Strategy](<../Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_Vendor_Exit_Strategy.md>) *(Data Architecture)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Effective Data Engineering in the Cloud World

