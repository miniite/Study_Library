---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Streaming-first_Data_Architecture"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["Apache_Flink", "Apache_Spark", "Apache_Kafka", "Databricks"]
---

# Streaming-first Data Architecture

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Flink](<../../../../TOOLS/Apache_Flink/_index.md>)
- [Apache Spark](<../../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Databricks](<../../../../TOOLS/Databricks/_index.md>)

## Practice questions

- [Intermediate] [A stakeholder wants 'real-time dashboards' but your architecture is nightly batch. How do you figure out whether they genuinely need streaming, and what's the smallest change that gets them closer?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11060_A_stakeholder_wants_real-time_dashboards_but_your_architectu.md>)
- [Deep] [What's the difference between streaming features and batch features for your model, and how does your choice of data architecture constrain which you can offer?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10659_What_s_the_difference_between_streaming_features_and_batch_f.md>)
- [Deep] [You're getting inconsistent metric values between your batch feature job (Spark) and a streaming version (Flink). How do you make them reconcile by construction?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10710_You_re_getting_inconsistent_metric_values_between_your_batch.md>)
- [Deep] [Explain the difference between event-time and processing-time and a time when confusing the two skewed your daily counts. How do you design around it?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11108_Explain_the_difference_between_event-time_and_processing-tim.md>)
- [Deep] [Explain the difference between a lambda and a kappa architecture in practical terms, and why maintaining two code paths for the same metric erodes trust in your numbers.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11139_Explain_the_difference_between_a_lambda_and_a_kappa_architec.md>)
- [Deep] [You're asked to eliminate a nightly 8-hour batch window that no longer fits the business's global operating hours. Walk me through converting a batch-dependent warehouse to continuously-loaded micro-batches or streaming, and where downstream consumers' assumptions about 'a complete day' break.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11020_You_re_asked_to_eliminate_a_nightly_8-hour_batch_window_that.md>)
- [Expert] [Your leadership wants near-real-time analytics but your architecture is nightly batch. Sequence the incremental evolution to streaming without a big-bang rewrite. Where do you start and what stays batch forever?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10589_Your_leadership_wants_near-real-time_analytics_but_your_arch.md>)
- [Expert] [Your batch feature pipeline (Spark) and streaming feature pipeline (Flink) compute the same feature and diverge, eroding model trust. Architect a batch/stream unification so the two paths reconcile by construction rather than by after-the-fact checks.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10763_Your_batch_feature_pipeline_Spark_and_streaming_feature_pipe.md>)

## Related concepts

- [Batch vs Streaming Cost Trade-offs](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs.md>) *(Batch vs Stream)*
- [Unified Batch and Streaming Semantics](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>) *(Batch vs Stream)*
- [Unified Batch and Streaming Platforms](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Platforms.md>) *(Batch vs Stream)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Streaming Is Different from Batch
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake

