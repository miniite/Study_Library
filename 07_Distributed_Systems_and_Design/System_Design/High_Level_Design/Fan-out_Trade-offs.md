---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["Apache_Kafka"]
---

# Fan-out Trade-offs

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [Your URL shortener's redirect p99 latency jumped from 20ms to 400ms in one region — walk through how you'd diagnose whether it's cache miss rate, DB hotspot, or replication lag.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29586_Your_URL_shortener_s_redirect_p99_latency_jumped_from_20ms_t.md>)
- [Deep] [A downstream push provider (APNs/FCM) starts returning 30% errors — how does your notification service degrade gracefully, retry, and avoid a retry storm that makes it worse?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29602_A_downstream_push_provider_APNs_FCM_starts_returning_30_erro.md>)
- [Deep] [Design backpressure for a message-driven service where the consumer is slower than the producer — compare bounded queues pausing consumption and load-shedding and how you signal producers to slow down.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29715_Design_backpressure_for_a_message-driven_service_where_the_c.md>)
- [Deep] [You're proposing to move from request-response to an async/queued model for a slow workflow — a skeptical PM asks how users still get instant feedback and how you handle failures they can't see; answer them.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29832_You_re_proposing_to_move_from_request-response_to_an_async_q.md>)
- [Deep] [Design an EdTech live-class platform delivering interactive classes to 100K concurrent students with video quizzes and doubt-solving — how do you architect the low-latency AV interactivity and recording so a class survives a regional network blip?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29959_Design_an_EdTech_live-class_platform_delivering_interactive_.md>)
- [Expert] [Design a distributed job/task scheduler that runs millions of cron-like and delayed jobs reliably — how do you guarantee a job runs exactly once at its scheduled time?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29622_Design_a_distributed_job_task_scheduler_that_runs_millions_o.md>)
- [Expert] [Lead the postmortem of a cascading outage where a 200ms latency bump in one service exhausted thread pools across a dozen callers — what systemic (not local) fixes do you mandate org-wide afterward?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29794_Lead_the_postmortem_of_a_cascading_outage_where_a_200ms_late.md>)
- [Expert] [You must design a guardrail-and-eval layer that sits in front of every LLM call org-wide — how do you architect prompt-injection defense PII redaction and output-safety checks without adding a second network hop of latency to every inference?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29884_You_must_design_a_guardrail-and-eval_layer_that_sits_in_fron.md>)

## Related concepts

- [Org-wide Backpressure](<../System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [Lag-based Consumer Autoscaling](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling.md>) *(Event Driven Architecture)*
- [Lock Granularity Design](<../Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>) *(Low Level Design)*

