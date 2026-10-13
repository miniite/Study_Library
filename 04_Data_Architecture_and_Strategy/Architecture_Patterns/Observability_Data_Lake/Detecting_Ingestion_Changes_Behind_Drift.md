---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Detecting_Ingestion_Changes_Behind_Drift"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake"
tools: ["AWS", "Apache_Kafka"]
---

# Detecting Ingestion Changes Behind Drift

**L5 Expert** · [↑ Observability Data Lake](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [How would you debug a situation where anomaly-detection jobs reading from the lake suddenly slowed 5x after a schema evolution event?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34401_How_would_you_debug_a_situation_where_anomaly-detection_jobs.md>)
- [Deep] [You're seeing duplicate log lines in the lake causing inflated error rates in an anomaly model. How do you find the source and dedupe safely?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34431_You_re_seeing_duplicate_log_lines_in_the_lake_causing_inflat.md>)
- [Deep] [How would you monitor the observability lake itself so it doesn't become an unmonitored single point of failure for all your AIOps?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34445_How_would_you_monitor_the_observability_lake_itself_so_it_do.md>)
- [Deep] [How do you detect and quantify sampling bias introduced by your trace sampling before it corrupts anomaly-model training data in the lake?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34453_How_do_you_detect_and_quantify_sampling_bias_introduced_by_y.md>)
- [Deep] [What's your plan for disaster recovery of the observability lake and how fast must it come back given AIOps depends on it during outages?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34480_What_s_your_plan_for_disaster_recovery_of_the_observability_.md>)
- [Deep] [What is your approach to platform team sizing and on-call for a lake that has become tier-0 infrastructure that all of AIOps depends on?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34521_What_is_your_approach_to_platform_team_sizing_and_on-call_fo.md>)
- [Deep] [What is your approach to disaster gamedays for the lake so you actually prove AIOps degrades gracefully when the lake loses a region or a compaction cluster?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34551_What_is_your_approach_to_disaster_gamedays_for_the_lake_so_y.md>)
- [Expert] [Walk me through a real incident where a lake ingestion or query problem caused an AIOps blind spot and how you detected and fixed it.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34493_Walk_me_through_a_real_incident_where_a_lake_ingestion_or_qu.md>)

## Related concepts

- [Data Quality Monitoring](<../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>) *(Data Observability)*
- [Reproducible Telemetry State](<../Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Reproducible_Telemetry_State.md>) *(Data Architecture)*
- [Streaming for AIOps](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Streaming_for_AIOps.md>) *(Streaming Architecture)*

