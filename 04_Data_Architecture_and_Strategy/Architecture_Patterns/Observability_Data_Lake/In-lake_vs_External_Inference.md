---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/In-lake_vs_External_Inference"
level: 5
type: "concept"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake"
tools: []
---

# In-lake vs External Inference

**L5 Expert** · [↑ Observability Data Lake](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [Walk me through how you would model unified metrics logs and traces in a data lake so that an AIOps correlation engine can join them at query time.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34398_Walk_me_through_how_you_would_model_unified_metrics_logs_and.md>)
- [Deep] [Walk me through instrumenting cost-per-query controls so a runaway AIOps model retraining job can't rack up a huge lake compute bill overnight.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34478_Walk_me_through_instrumenting_cost-per-query_controls_so_a_r.md>)
- [Deep] [How would you decide the boundary between what lives in the observability lake versus what stays in purpose-built systems like a TSDB APM and SIEM at org scale?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34505_How_would_you_decide_the_boundary_between_what_lives_in_the_.md>)
- [Deep] [How do you make the case for separating storage and compute at org scale when a VP argues a coupled appliance would be simpler and faster to operate?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34516_How_do_you_make_the_case_for_separating_storage_and_compute_.md>)
- [Deep] [How do you decide when to run AIOps model inference inside the lake platform versus as an external service and what are the org and reliability implications of each?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34547_How_do_you_decide_when_to_run_AIOps_model_inference_inside_t.md>)
- [Deep] [How do you evaluate the ROI of investing in a feature store over the lake for AIOps versus letting each model pipeline compute features ad hoc?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34564_How_do_you_evaluate_the_ROI_of_investing_in_a_feature_store_.md>)
- [Deep] [What is your approach to defining what good looks like for the lake's query performance and how do you keep SLOs honest as data volume 10x's over three years?](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34583_What_is_your_approach_to_defining_what_good_looks_like_for_t.md>)
- [Expert] [Make the build-versus-buy case to a CTO for an in-house Iceberg lake versus a Datadog or Splunk contract at a 2000-engineer company and quantify the crossover point.](<../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Q-34498_Make_the_build-versus-buy_case_to_a_CTO_for_an_in-house_Iceb.md>)

## Related concepts

- [Open vs Proprietary Formats](<../../Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats.md>) *(Data Platform Strategy)*
- [Choosing an Event Backbone](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>) *(Distributed Systems Core)*
- [Streaming for AIOps](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Streaming_for_AIOps.md>) *(Streaming Architecture)*

