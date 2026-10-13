---
id: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Autoscaling_Metrics"
level: 4
type: "concept"
topic: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals"
tools: ["AWS"]
---

# Autoscaling Metrics

**L4 Advanced** · [↑ Cloud Fundamentals](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Basic] [What is auto-scaling and what metric might trigger adding more model-serving instances during peak traffic?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09342_What_is_auto-scaling_and_what_metric_might_trigger_adding_mo.md>)
- [Intermediate] [An engineer set an autoscaling target-tracking policy on 50% CPU but the fleet oscillates between scale-out and scale-in every few minutes. What tuning of cooldowns and target value would you apply and why?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09083_An_engineer_set_an_autoscaling_target-tracking_policy_on_50_.md>)
- [Deep] [How would you decide the autoscaling metric for an inference service whose real pressure is queued request depth rather than CPU or GPU utilization?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08201_How_would_you_decide_the_autoscaling_metric_for_an_inference.md>)
- [Deep] [How do you decide the right autoscaling metric for a service whose load is measured in queue depth rather than CPU?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08583_How_do_you_decide_the_right_autoscaling_metric_for_a_service.md>)
- [Deep] [How would you design an autoscaling policy that reacts to a custom business metric like orders-per-minute instead of CPU, and what publishing latency issues would you watch for?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08677_How_would_you_design_an_autoscaling_policy_that_reacts_to_a_.md>)
- [Deep] [How would you design an autoscaling group that scales on queue depth but also respects a downstream database's connection ceiling so scaling out doesn't self-DDoS the DB?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08737_How_would_you_design_an_autoscaling_group_that_scales_on_que.md>)
- [Deep] [How do you decide the right autoscaling metric for a worker fleet whose real pressure is queue backlog rather than CPU, and what publishing-lag pitfalls would you watch for?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09105_How_do_you_decide_the_right_autoscaling_metric_for_a_worker_.md>)
- [Deep] [Your inference GPU sits at 15% utilization because requests trickle in one at a time. How would you use dynamic batching and cloud autoscaling to raise utilization without hurting latency SLOs?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09419_Your_inference_GPU_sits_at_15_utilization_because_requests_t.md>)

## Related concepts

- [Azure Functions and Cosmos DB Tuning](<../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/_index.md>) *(Azure Platform)*
- [Org-wide Backpressure](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [Admission Control and Load Shedding](<../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Admission_Control_and_Load_Shedding.md>) *(Python Language)*

