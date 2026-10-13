---
id: "03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream"
tools: ["Apache_Kafka", "Apache_Flink", "Apache_Spark"]
---

# Batch vs Streaming Cost Trade-offs

**L4 Advanced** · [↑ Batch vs Stream](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Flink](<../../../TOOLS/Apache_Flink/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Basic] [A dashboard shows 'orders in the last 5 minutes'. Would batch or stream be more appropriate and why?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04231_A_dashboard_shows_orders_in_the_last_5_minutes_Would_batch_o.md>)
- [Intermediate] [In batch, how do you decide how often to run the job (hourly, daily, weekly)?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04237_In_batch_how_do_you_decide_how_often_to_run_the_job_hourly_d.md>)
- [Intermediate] [How would you explain to a stakeholder why real-time data costs more to deliver than daily data?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04309_How_would_you_explain_to_a_stakeholder_why_real-time_data_co.md>)
- [Intermediate] [How do you decide the micro-batch trigger interval when SLA is 'under 2 minutes' but the cluster idles most of the time between batches?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04334_How_do_you_decide_the_micro-batch_trigger_interval_when_SLA_.md>)
- [Deep] [What is the difference between true streaming (event-at-a-time) and micro-batch streaming?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04295_What_is_the_difference_between_true_streaming_event-at-a-tim.md>)
- [Deep] [When events can arrive hours late, do you extend the streaming window or push corrections through a batch job? Walk the decision.](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04276_When_events_can_arrive_hours_late_do_you_extend_the_streamin.md>)
- [Deep] [What's your framework, in an interview, for turning a vague 'we want real-time analytics' ask into a concrete batch-vs-stream design decision?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04298_What_s_your_framework_in_an_interview_for_turning_a_vague_we.md>)
- [Deep] [A batch job and its streaming equivalent produce different daily counts. Which one do you trust to debug the other, and why?](<../../../PRACTICE/03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Q-04347_A_batch_job_and_its_streaming_equivalent_produce_different_d.md>)

## Related concepts

- [Streaming-first Data Architecture](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Streaming-first_Data_Architecture.md>) *(Data Architecture)*
- [Batch vs Streaming Features](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/_index.md>) *(Pipeline Orchestration)*
- [Unified Batch and Streaming Semantics](<Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>) *(Batch vs Stream)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Time (Semantics) Won't Wait

