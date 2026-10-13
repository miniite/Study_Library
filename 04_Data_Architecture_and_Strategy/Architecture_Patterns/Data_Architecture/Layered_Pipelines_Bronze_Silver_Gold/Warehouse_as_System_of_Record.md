---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Warehouse_as_System_of_Record"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["Apache_Spark", "AWS", "dbt"]
---

# Warehouse as System of Record

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../../TOOLS/Apache_Spark/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [dbt](<../../../../TOOLS/dbt/_index.md>)

## Practice questions

- [Basic] [What's the practical difference between a data lake, a warehouse, and a lakehouse from the perspective of an ML engineer who needs both raw logs and clean aggregates?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10651_What_s_the_practical_difference_between_a_data_lake_a_wareho.md>)
- [Intermediate] [Explain the trade-off of transforming raw data into features inside the warehouse (SQL) versus in a Spark job for your particular feature set.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10724_Explain_the_trade-off_of_transforming_raw_data_into_features.md>)
- [Deep] [Explain schema-on-read versus schema-on-write with an example from your own pipelines. When has schema-on-read bitten you and how did you contain it?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11058_Explain_schema-on-read_versus_schema-on-write_with_an_exampl.md>)
- [Deep] [How do you decide what belongs in the operational plane (OLTP/event log) vs the analytical plane (lake/warehouse), and how do you keep the coupling between them from becoming a distributed monolith?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10563_How_do_you_decide_what_belongs_in_the_operational_plane_OLTP.md>)
- [Deep] [Argue whether an org should standardize on ELT (transform in-warehouse) or ETL (transform before load) as its paradigm. What does each imply for cost, governance, and where data engineering talent must sit?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10604_Argue_whether_an_org_should_standardize_on_ELT_transform_in-.md>)
- [Deep] [Argue whether the warehouse should ever be system-of-record for any feature, or purely a derived analytical copy. What goes wrong for ML when teams start treating a derived feature table as authoritative for operational decisions?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10835_Argue_whether_the_warehouse_should_ever_be_system-of-record_.md>)
- [Deep] [Your ELT-in-warehouse approach means all transformation logic lives in 8000 dbt models with a 6-hour DAG. As principal, how do you decide what should move out of the warehouse (to Spark/streaming) versus stay, and how do you prevent the warehouse from becoming an expensive general-purpose compute engine?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10975_Your_ELT-in-warehouse_approach_means_all_transformation_logi.md>)
- [Deep] [You're asked whether to standardize on a single cloud warehouse SQL dialect or maintain a translation/abstraction layer so the org isn't locked to one vendor's SQL. What does an abstraction like SQLGlot/dbt-adapters actually cost you in features, and when is deliberate lock-in the right call?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11005_You_re_asked_whether_to_standardize_on_a_single_cloud_wareho.md>)

## Related concepts

- [Data Warehouse Fundamentals](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Data Vault](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Choosing a Database for the Workload](<../../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Six Dimensions for Picking an Analytical Data Warehouse
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Tools Don't Matter, Patterns and Practices Do
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — IntroduCtIon to analytIC data In a transaCtIonal database
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle

