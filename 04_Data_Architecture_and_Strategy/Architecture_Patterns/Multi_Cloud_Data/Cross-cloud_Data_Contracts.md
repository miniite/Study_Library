---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data"
tools: ["AWS", "Google_Cloud", "Azure", "Snowflake"]
---

# Cross-cloud Data Contracts

**L5 Expert** · [↑ Multi Cloud Data](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Google Cloud](<../../../TOOLS/Google_Cloud/_index.md>)
- [Azure](<../../../TOOLS/Azure/_index.md>)
- [Snowflake](<../../../TOOLS/Snowflake/_index.md>)

## Practice questions

- [Deep] [When a business unit demands GCP for its ML tooling while your platform standard is AWS, how do you draw the line between a sanctioned second cloud and shadow IT you must shut down?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32900_When_a_business_unit_demands_GCP_for_its_ML_tooling_while_yo.md>)
- [Deep] [A data product needs to join a table in BigQuery with one in Snowflake for a single report — what's the least-bad architecture, and why is 'just federate it' usually the wrong default?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32928_A_data_product_needs_to_join_a_table_in_BigQuery_with_one_in.md>)
- [Deep] [You need to give a data-science team read access to production data that spans two clouds without copying it into notebooks — design the governed access pattern and its audit trail.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32941_You_need_to_give_a_data-science_team_read_access_to_producti.md>)
- [Deep] [How do you handle the divergence in supported SQL dialects and functions when the same transformation must run in BigQuery and Snowflake — do you abstract with dbt or accept forks?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32964_How_do_you_handle_the_divergence_in_supported_SQL_dialects_a.md>)
- [Deep] [Design the change-management process for a shared cross-cloud contract when a producer in cloud A wants a breaking change that impacts consumers in cloud B — who has veto and how is it phased?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32994_Design_the_change-management_process_for_a_shared_cross-clou.md>)
- [Expert] [How do you design lineage that survives a pipeline that reads from S3, transforms in Databricks, and lands in BigQuery — where does OpenLineage instrumentation break across cloud boundaries?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32902_How_do_you_design_lineage_that_survives_a_pipeline_that_read.md>)
- [Expert] [You inherit two independent data platforms from a merger — one on Azure Synapse and one on Snowflake-on-AWS. Lay out a 24-month convergence strategy and the sequencing of what you unify first.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32908_You_inherit_two_independent_data_platforms_from_a_merger_one.md>)
- [Expert] [A subsidiary in China must run on a local cloud with no cross-border transfer — how does that partition change your global catalog, lineage and metrics-layer design?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Q-32936_A_subsidiary_in_China_must_run_on_a_local_cloud_with_no_cros.md>)

## Related concepts

- [Platform-level Data Quality](<../../Platform_Strategy/Data_Platform_Strategy/Platform-level_Data_Quality.md>) *(Data Platform Strategy)*
- [Data Lifecycle and Retention](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Data_Lifecycle_and_Retention.md>) *(Cloud Fundamentals)*
- [Audit and Compliance Platforms](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>) *(High Level Design)*

