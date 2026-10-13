---
id: "TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_Beyond_Kafka"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka", "PostgreSQL"]
---

# Exactly Once Beyond Kafka

**L4 Advanced** · [↑ Kafka Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Exactly Once to External Sinks](<Exactly_Once_to_External_Sinks.md>)

## Implementations (tools)

- [Apache Kafka](<../../_index.md>)
- [PostgreSQL](<../../../PostgreSQL/_index.md>)

## Practice questions

- [Intermediate] [What does 'at-least-once' delivery mean in Kafka and when does it occur?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30524_What_does_at-least-once_delivery_mean_in_Kafka_and_when_does.md>)
- [Intermediate] [Your dbt models run on the warehouse tables that a Kafka sink lands into; how do you make dbt runs consistent given the underlying table is being continuously appended?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31099_Your_dbt_models_run_on_the_warehouse_tables_that_a_Kafka_sin.md>)
- [Deep] [What does 'exactly-once' loosely promise and why is it hard to truly get from Kafka all the way into your warehouse?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31069_What_does_exactly-once_loosely_promise_and_why_is_it_hard_to.md>)
- [Deep] [Why can't you rely on Kafka's exactly-once transactions to also make your warehouse insert exactly-once, and how do you bridge the gap?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31079_Why_can_t_you_rely_on_Kafka_s_exactly-once_transactions_to_a.md>)
- [Deep] [Your team wants exactly-once from Kafka to Postgres. Explain why the Kafka transaction alone is insufficient and how you'd make the DB write idempotent instead.](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30735_Your_team_wants_exactly-once_from_Kafka_to_Postgres_Explain_.md>)
- [Deep] [You want at-least-once from Kafka to a warehouse using Spark; where exactly is the offset committed relative to the write, and how do you make the sink idempotent to compensate?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31093_You_want_at-least-once_from_Kafka_to_a_warehouse_using_Spark.md>)
- [Expert] [How would you achieve exactly-once processing from Kafka into a downstream database?](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30604_How_would_you_achieve_exactly-once_processing_from_Kafka_int.md>)
- [Expert] [Your remediation orchestrator uses exactly-once from Kafka to Postgres for the action-audit log. Explain why the Kafka transaction alone isn't enough and how you make the DB write idempotent.](<../../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30984_Your_remediation_orchestrator_uses_exactly-once_from_Kafka_t.md>)

## Related concepts

- [End to End Exactly Once](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once.md>) *(Big Data Processing)*
- [Consuming Untrusted Streams](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>) *(Streaming Architecture)*
- [Tiered Storage for Logs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>) *(Event Driven Architecture)*

