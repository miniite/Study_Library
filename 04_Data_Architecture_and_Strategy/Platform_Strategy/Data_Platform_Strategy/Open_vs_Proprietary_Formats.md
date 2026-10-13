---
id: "04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy"
tools: ["AWS", "Apache_Spark", "Snowflake", "Kubernetes"]
---

# Open vs Proprietary Formats

**L5 Expert** · [↑ Data Platform Strategy](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Snowflake](<../../../TOOLS/Snowflake/_index.md>)
- [Kubernetes](<../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Deep] [Why not just run everything on Snowflake and be done with it — what forces you to introduce a second compute engine into the platform and how do you avoid two-engine sprawl?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Q-13750_Why_not_just_run_everything_on_Snowflake_and_be_done_with_it.md>)
- [Deep] [How do you make a strategic argument for or against real-time when 90% of business questions are answered by daily data — where does streaming genuinely earn its complexity cost?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Q-13772_How_do_you_make_a_strategic_argument_for_or_against_real-tim.md>)
- [Deep] [How do you evaluate ClickHouse/Druid/Pinot as a platform-standard real-time OLAP layer vs pushing that load onto the warehouse — what workload signature tells you a dedicated speed layer is warranted?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Q-13829_How_do_you_evaluate_ClickHouse_Druid_Pinot_as_a_platform-sta.md>)
- [Deep] [Probe: you sized the platform for the analytics use-case, but now product wants it in the request path with 50ms SLAs — is that the same platform or a different one? How do you avoid one workload's needs distorting the whole strategy?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Q-13842_Probe_you_sized_the_platform_for_the_analytics_use-case_but_.md>)
- [Expert] [How would you sequence a migration from a proprietary ETL tool (Informatica/Ab Initio) to code-first pipelines for 5000 existing jobs — big-bang, strangler-fig, or freeze-and-forward?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Q-13770_How_would_you_sequence_a_migration_from_a_proprietary_ETL_to.md>)
- [Expert] [What's your take on 'zero-copy' data sharing (Snowflake shares / Delta Sharing / Iceberg REST) as a strategic capability — where does it change the org boundary and where does it introduce governance holes?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Q-13780_What_s_your_take_on_zero-copy_data_sharing_Snowflake_shares_.md>)
- [Expert] [You have to present a 'do nothing' vs 'invest' comparison for platform modernization to a skeptical board — how do you quantify the cost of inaction and the risk of over-investing?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Q-13790_You_have_to_present_a_do_nothing_vs_invest_comparison_for_pl.md>)
- [Expert] [How do you decide the platform's position on open table formats vs staying inside a vendor's proprietary storage — what's the real cost of 'openness' when the vendor's optimizer only shines on native?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Q-13811_How_do_you_decide_the_platform_s_position_on_open_table_form.md>)

## Related concepts

- [Choosing an Event Backbone](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>) *(Distributed Systems Core)*
- [Warehouse as System of Record](<../../Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*
- [Database Vendor and Platform Choices](<../../../01_Databases_and_SQL/Database_Strategy/Vendor_and_Tooling/Database_Vendor_and_Platform_Choices.md>) *(Vendor and Tooling)*

## References

- *Building Data Integration Solutions* — Jay Borthen — Data Integration Tool Options
- *Deciphering Data Architectures* — James Serra — Technologies
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle

