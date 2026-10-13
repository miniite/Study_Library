---
id: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Sync_vs_Async_Processing"
level: 4
type: "concept"
topic: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals"
tools: ["AWS"]
---

# Sync vs Async Processing

**L4 Advanced** · [↑ Cloud Fundamentals](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Basic] [Why might you put a message queue between your web app and a slow LLM-processing worker in the cloud?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08371_Why_might_you_put_a_message_queue_between_your_web_app_and_a.md>)
- [Intermediate] [What is a message queue and why would you use one between two services in the cloud?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08658_What_is_a_message_queue_and_why_would_you_use_one_between_tw.md>)
- [Intermediate] [Explain how you would use a cloud message queue between an ingestion service and a feature computation worker so bursty upstream data does not overwhelm the workers.](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08169_Explain_how_you_would_use_a_cloud_message_queue_between_an_i.md>)
- [Deep] [Why is FIFO ordering harder to scale than best-effort ordering in a distributed queue?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08406_Why_is_FIFO_ordering_harder_to_scale_than_best-effort_orderi.md>)
- [Deep] [How do you decide the batch size and concurrency for a consumer draining a queue, balancing throughput against downstream pressure and latency?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08620_How_do_you_decide_the_batch_size_and_concurrency_for_a_consu.md>)
- [Deep] [How would you decide what to make synchronous versus asynchronous in a checkout flow so the user gets fast confirmation but nothing critical is lost?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08664_How_would_you_decide_what_to_make_synchronous_versus_asynchr.md>)
- [Deep] [How would you decide between websockets on a dedicated fleet versus a managed real-time service for a chat feature expecting 100k concurrent connections?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08733_How_would_you_decide_between_websockets_on_a_dedicated_fleet.md>)
- [Deep] [You need to set a company-wide policy on synchronous versus asynchronous inter-service calls so teams stop building fragile blocking chains. How do you enforce it without dictating every integration?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08895_You_need_to_set_a_company-wide_policy_on_synchronous_versus_.md>)

## Related concepts

- [Queue vs Synchronous Calls](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Queue_vs_Synchronous_Calls.md>) *(System Design Core)*
- [Fan-out Trade-offs](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Lag-based Consumer Autoscaling](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling.md>) *(Event Driven Architecture)*

## References

- *Communication Patterns* — Jacqui Read — Remote Principles
- *Fundamentals of Software Architecture (2nd ed.)* — Mark Richards, Neal Ford — Synchronous Communication

