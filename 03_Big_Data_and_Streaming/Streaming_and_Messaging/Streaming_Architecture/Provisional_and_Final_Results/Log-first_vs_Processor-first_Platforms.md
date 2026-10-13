---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Log-first_vs_Processor-first_Platforms"
level: 5
type: "deep-dive"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture"
tools: ["Apache_Kafka", "Apache_Flink", "AWS", "Kubernetes"]
---

# Log-first vs Processor-first Platforms

**L5 Expert** · [↑ Streaming Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Flink](<../../../../TOOLS/Apache_Flink/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [Why might you choose Kafka Streams over Flink for a mid-scale AIOps alert-enrichment pipeline, and what would make you regret that choice as the team grows?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38134_Why_might_you_choose_Kafka_Streams_over_Flink_for_a_mid-scal.md>)
- [Deep] [A team wants to add heavy per-event enrichment via synchronous calls to a downstream service inside a streaming operator. Explain why that's an anti-pattern at scale and design the async enrichment architecture you'd mandate instead.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38075_A_team_wants_to_add_heavy_per-event_enrichment_via_synchrono.md>)
- [Deep] [Design how the streaming platform exposes real-time SLIs to hundreds of teams as a product feature rather than each team hand-rolling their own SLI computation. What's the abstraction, and how do you keep it correct when teams define exotic SLIs?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38242_Design_how_the_streaming_platform_exposes_real-time_SLIs_to_.md>)
- [Deep] [How do you architect for the case where the AIOps model needs both streaming features (live) and slowly-changing enrichment (topology, ownership, deploy metadata) joined consistently? Where does the slowly-changing side live and how do you avoid a stale-join incident?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38298_How_do_you_architect_for_the_case_where_the_AIOps_model_need.md>)
- [Expert] [Compare Flink, Kafka Streams, and Spark Structured Streaming as a platform bet for the next five years across operational model, state handling, and team ergonomics. Which do you standardize on and how do you handle the teams already invested in the other two?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38036_Compare_Flink_Kafka_Streams_and_Spark_Structured_Streaming_a.md>)
- [Expert] [Compare designing your streaming platform around a log-first (Kafka/Pulsar) model versus a stream-processor-first (Flink managed streams) model. What second-order consequences on team boundaries and evolvability does each choice lock in?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38086_Compare_designing_your_streaming_platform_around_a_log-first.md>)
- [Expert] [Design a streaming architecture for real-time ML inference where features arrive as streams and the model is updated frequently. How do you version features and models together, and how do you prevent a bad model deploy from silently poisoning downstream decisions?](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38113_Design_a_streaming_architecture_for_real-time_ML_inference_w.md>)
- [Expert] [A principal probe: 'You picked Flink as the org standard for stream processing. In five years, what's the most likely reason a successor architect rips it out?' Give an honest answer about where your bet is most fragile.](<../../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Q-38234_A_principal_probe_You_picked_Flink_as_the_org_standard_for_s.md>)

## Related concepts

- [Choosing an Event Backbone](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>) *(Distributed Systems Core)*
- [Unified Batch and Streaming Platforms](<../../../Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Platforms.md>) *(Batch vs Stream)*
- [Shared Pipeline Platforms](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/Shared_Pipeline_Platforms.md>) *(Pipeline Orchestration)*

