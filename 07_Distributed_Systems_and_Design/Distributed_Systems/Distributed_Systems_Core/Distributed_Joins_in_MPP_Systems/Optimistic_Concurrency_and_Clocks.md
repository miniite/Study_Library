---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Optimistic_Concurrency_and_Clocks"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["Apache_Spark", "Apache_Kafka", "AWS", "Apache_Flink", "Databricks"]
---

# Optimistic Concurrency and Clocks

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Apache Flink](<../../../../TOOLS/Apache_Flink/_index.md>)
- [Databricks](<../../../../TOOLS/Databricks/_index.md>)

## Practice questions

- [Intermediate] [Your evaluation dataset lives in a warehouse and a query returns slightly different rows run to run, making eval scores non-reproducible. Explain how snapshot isolation or dataset versioning like Iceberg time-travel fixes this and why immutable eval sets matter.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23160_Your_evaluation_dataset_lives_in_a_warehouse_and_a_query_ret.md>)
- [Intermediate] [Explain the difference between at-most-once at-least-once and exactly-once semantics with a concrete data-ingestion example of each and which you'd default to for a metrics-ingestion pipeline versus a billing events pipeline.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24323_Explain_the_difference_between_at-most-once_at-least-once_an.md>)
- [Deep] [You have to fan a write out to N downstream systems and they must all eventually reflect it, but some are flaky. Compare a queue-per-consumer, an outbox+CDC, and a workflow engine for this fan-out.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23296_You_have_to_fan_a_write_out_to_N_downstream_systems_and_they.md>)
- [Deep] [Your Airflow scheduler fell behind and DAG runs piled up so a downstream task ran on data an upstream hadn't produced yet. Explain how scheduler distribution executor choice and sensors/dependencies prevent this out-of-order execution.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24272_Your_Airflow_scheduler_fell_behind_and_DAG_runs_piled_up_so_.md>)
- [Deep] [Your nightly query reads a table that another job is actively appending to and you get slightly different results run-to-run. Explain snapshot isolation on a lakehouse table and how pinning a snapshot id fixes non-reproducibility.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24294_Your_nightly_query_reads_a_table_that_another_job_is_activel.md>)
- [Deep] [A gray failure: a Spark executor passes its health check but silently drops or mis-writes 2% of records. Explain how you'd catch this with row-count reconciliation and differential checks rather than trusting the executor's self-report.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24341_A_gray_failure_a_Spark_executor_passes_its_health_check_but_.md>)
- [Expert] [Explain exactly-once delivery vs exactly-once processing. Why is exactly-once delivery generally impossible over a network, and how does exactly-once processing still get achieved in practice?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23242_Explain_exactly-once_delivery_vs_exactly-once_processing_Why.md>)
- [Expert] [Explain how Delta Lake and Iceberg differ in their concurrency-control mechanism for concurrent writers and how each handles a compaction running at the same time as an append without corrupting a reader's snapshot.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24261_Explain_how_Delta_Lake_and_Iceberg_differ_in_their_concurren.md>)

## Related concepts

- [Cross-service Invalidation Races](<../../Concurrency_Patterns/Cross-service_Invalidation_Races.md>) *(Concurrency Patterns)*
- [Exactly Once Spark Writes](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Exactly_Once_Spark_Writes.md>) *(Pipeline Orchestration)*
- [Table Format Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>) *(Table Formats)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Detecting Concurrent Writes › Capturing the happens-before relationship
- *Designing Data-Intensive Applications* — Martin Kleppmann — Single-Object and Multi-Object Operations › Single-object writes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Sequence Number Ordering › Timestamp ordering is not sufficient
- *Designing Data-Intensive Applications* — Martin Kleppmann — Don't confuse 2PC and 2PL › Coordinator failure
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Warning
- *Hadoop: The Definitive Guide* — Tom White — Flume › Delivery Guarantees
- *Hadoop: The Definitive Guide* — Tom White — Sqoop › Exports and Transactionality
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Fault Tolerance
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Exactly-once feeling
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Problem
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Building Native Applications with Delta Lake › S3DynamoDBLogStore
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Building Native Applications with Delta Lake › Concurrency with S3-compatible stores
- *Think Distributed Systems* — Dominik Tornow — Message delivery and processing › 4.4 Exactly-once processing semantics
- *Think Distributed Systems* — Dominik Tornow — Durable executions › 11.3 The concept of failure-transparent recovery
- *Think Distributed Systems* — Dominik Tornow — Durable executions › 11.5.2 Platform-level implementation: Durable execution
- *Database Internals* — Alex Petrov — Introduction and Overview › Fair-loss link
- *Database Internals* — Alex Petrov — Chapter 14. Consensus › Atomic Broadcast

