---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Warehouse_Vendor_Exit_Strategy"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS", "Hadoop", "Azure"]
---

# Warehouse Vendor Exit Strategy

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Hadoop](<../../../../TOOLS/Hadoop/_index.md>)
- [Azure](<../../../../TOOLS/Azure/_index.md>)

## Practice questions

- [Intermediate] [You inherit an S3 bucket with 2 years of Parquet files, no catalog, no partition scheme, and no owner. Give me your first two weeks of work to make it queryable and trustworthy.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11057_You_inherit_an_S3_bucket_with_2_years_of_Parquet_files_no_ca.md>)
- [Deep] [How do you architect CDC from hundreds of operational databases into a lake so ML teams get fresh features, and at what point does CDC-as-backbone become a reliability liability that degrades model freshness org-wide?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10766_How_do_you_architect_CDC_from_hundreds_of_operational_databa.md>)
- [Deep] [How do you architect the boundary between a document store (MongoDB/DynamoDB) as system of record and a columnar analytical lake, given schema flexibility on the operational side? Where does schema-on-read at the lake become a liability when the document shape drifts silently across releases?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10956_How_do_you_architect_the_boundary_between_a_document_store_M.md>)
- [Expert] [You inherit a data swamp: an S3 lake with no catalog, no ownership, and 3PB of unknown provenance. Give me the 90-day and 12-month plan to make it governed without freezing every downstream consumer.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10537_You_inherit_a_data_swamp_an_S3_lake_with_no_catalog_no_owner.md>)
- [Expert] [Your streaming platform (Kafka/Kinesis) and your lake are owned by different teams with conflicting retention and schema-evolution policies. As principal, how do you architect the boundary and resolve the org conflict?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10582_Your_streaming_platform_Kafka_Kinesis_and_your_lake_are_owne.md>)
- [Expert] [How do you architect the metadata/control plane so it doesn't become a single point of failure for the entire analytical estate? What's the availability target for the catalog and how do you decouple it from data-plane operations?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10635_How_do_you_architect_the_metadata_control_plane_so_it_doesn_.md>)
- [Expert] [A regulator requires you to reproduce the exact training dataset for any credit model on demand for 7 years. Architect the immutable snapshotting and retention layer across a lakehouse without unbounded storage growth.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10753_A_regulator_requires_you_to_reproduce_the_exact_training_dat.md>)
- [Expert] [You inherit a data swamp: an S3 feature lake, no catalog, no ownership, 5PB of unknown provenance feeding live models. Give the 90-day and 12-month plan to govern it without freezing any model's retraining.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10801_You_inherit_a_data_swamp_an_S3_feature_lake_no_catalog_no_ow.md>)

## Related concepts

- [Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>) *(High Level Design)*
- [Region Failure Resilience](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Region_Failure_Resilience.md>) *(Architecture Strategy)*
- [Lakehouse Fundamentals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*

## References

- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Accelerating Unity Catalog Adoption
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Key takeaways
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Part IGetting Started with Azure Data Factory and Mapping Data Flows

