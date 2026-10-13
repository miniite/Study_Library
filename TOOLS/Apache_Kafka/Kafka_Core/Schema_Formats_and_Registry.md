---
id: "TOOLS/Apache_Kafka/Kafka_Core/Schema_Formats_and_Registry"
level: 4
type: "concept"
topic: "TOOLS/Apache_Kafka/Kafka_Core"
tools: ["Apache_Kafka"]
---

# Schema Formats and Registry

**L4 Advanced** · [↑ Kafka Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../_index.md>)

## Practice questions

- [Basic] [Why might a data engineer prefer JSON on a Kafka topic when starting out but move to Avro later?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30995_Why_might_a_data_engineer_prefer_JSON_on_a_Kafka_topic_when_.md>)
- [Intermediate] [You joined a team where events use Avro with a Schema Registry. How does that help keep the warehouse loads reliable over time?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31034_You_joined_a_team_where_events_use_Avro_with_a_Schema_Regist.md>)
- [Deep] [What is the Schema Registry and what problem does it solve in a Kafka ecosystem?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30547_What_is_the_Schema_Registry_and_what_problem_does_it_solve_i.md>)
- [Deep] [Design a mechanism to detect and quarantine a producer that starts sending events with a corrupt or unregistered schema before consumers are affected.](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30485_Design_a_mechanism_to_detect_and_quarantine_a_producer_that_.md>)
- [Deep] [How does the Schema Registry's subject naming strategy (topic vs record vs topic-record) affect a topic that legitimately carries multiple event types for a data platform?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-31117_How_does_the_Schema_Registry_s_subject_naming_strategy_topic.md>)
- [Deep] [You must convince a skeptical VP of Engineering that adopting event-driven architecture on Kafka is worth the operational tax. What business-outcome argument evidence and phased proof-of-value would you present?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30499_You_must_convince_a_skeptical_VP_of_Engineering_that_adoptin.md>)
- [Expert] [Conway's law is fighting your topic design: teams keep creating chatty cross-team topics that mirror the org chart. How would you reshape team topologies and topic ownership so the event model reflects business domains not reporting lines?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30498_Conway_s_law_is_fighting_your_topic_design_teams_keep_creati.md>)
- [Expert] [You're the tie-breaker in a cross-team dispute: the platform team wants one canonical event schema per domain, the product teams want their own topic-per-use-case. As principal, how do you adjudicate, and what principle governs when to unify versus fragment topics?](<../../../PRACTICE/TOOLS/Apache_Kafka/Kafka_Core/Q-30852_You_re_the_tie-breaker_in_a_cross-team_dispute_the_platform_.md>)

## Related concepts

- [Schema IDs and Compaction](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Schema_IDs_and_Compaction.md>) *(Schema Registry)*
- [Topic and Schema Design for Pipelines](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>) *(Big Data Processing)*
- [Phased Schema Rollouts](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Phased_Schema_Rollouts.md>) *(Schema Registry)*

