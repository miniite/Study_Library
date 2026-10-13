---
id: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Exactly_Once_Scheduled_Jobs"
level: 4
type: "concept"
topic: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals"
tools: ["AWS", "Redis"]
---

# Exactly Once Scheduled Jobs

**L4 Advanced** · [↑ Cloud Fundamentals](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [What is a cron job in the cloud and how would you run a scheduled task reliably?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09279_What_is_a_cron_job_in_the_cloud_and_how_would_you_run_a_sche.md>)
- [Intermediate] [How would you make sure a scheduled cloud job that failed last night actually retries instead of silently skipping?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09078_How_would_you_make_sure_a_scheduled_cloud_job_that_failed_la.md>)
- [Deep] [What is idempotency and why is it important for retried cloud API calls?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09190_What_is_idempotency_and_why_is_it_important_for_retried_clou.md>)
- [Deep] [Your training job's checkpoints to object storage are slowing the run because writes block the training loop. How would you make checkpointing asynchronous and durable?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08187_Your_training_job_s_checkpoints_to_object_storage_are_slowin.md>)
- [Deep] [How would you design an idempotency layer for a payment webhook that a provider may deliver two or three times with slightly different timestamps?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08568_How_would_you_design_an_idempotency_layer_for_a_payment_webh.md>)
- [Deep] [Your batch job on spot instances keeps getting interrupted mid-work and restarting from scratch. How do you checkpoint progress so interruptions cost minutes not hours?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08684_Your_batch_job_on_spot_instances_keeps_getting_interrupted_m.md>)
- [Deep] [Your batch job on spot instances restarts from scratch on every interruption, wasting hours. How would you add checkpointing so an interruption costs minutes, using only cloud storage primitives?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09158_Your_batch_job_on_spot_instances_restarts_from_scratch_on_ev.md>)
- [Expert] [Your team wants exactly-once processing from a queue. Explain why that is effectively impossible end to end and how you would achieve effectively-once with idempotency.](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08575_Your_team_wants_exactly-once_processing_from_a_queue_Explain.md>)

## Related concepts

- [Idempotent Order APIs](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*
- [Idempotent Ingestion](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>) *(Distributed Systems Core)*
- [Compensating Transactions](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>) *(Low Level Design)*

