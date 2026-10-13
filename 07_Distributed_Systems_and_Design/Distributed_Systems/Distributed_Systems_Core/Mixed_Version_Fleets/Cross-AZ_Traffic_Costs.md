---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Cross-AZ_Traffic_Costs"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["AWS"]
---

# Cross-AZ Traffic Costs

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [You deploy across three AZs behind an ALB but a zonal outage still caused a full outage. Walk me through how cross-zone load balancing zonal shift and unhealthy-host ejection interact and where the design failed.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23420_You_deploy_across_three_AZs_behind_an_ALB_but_a_zonal_outage.md>)
- [Intermediate] [Explain how you'd run a game day to validate that your multi-AZ RDS failover actually meets its RTO. What do you inject what do you measure and what commonly-untested assumption tends to break?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23448_Explain_how_you_d_run_a_game_day_to_validate_that_your_multi.md>)
- [Deep] [Your service's cross-AZ data transfer bill exceeded compute cost. Explain the design decisions that create this and how you'd add AZ-affinity routing without sacrificing the AZ-failure resilience you built.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23473_Your_service_s_cross-AZ_data_transfer_bill_exceeded_compute_.md>)
- [Deep] [You need cross-AZ GPU-to-GPU communication for training but the data-transfer cost is huge. Explain how AZ placement affects both training-network cost and fault tolerance, and how you'd co-locate a training job in one AZ without losing resilience.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23702_You_need_cross-AZ_GPU-to-GPU_communication_for_training_but_.md>)
- [Deep] [You run detectors across three AZs behind a load balancer, but a zonal outage still caused a full detection gap. Walk me through how cross-zone balancing, unhealthy-host ejection, and zonal shift interact and where the design failed to isolate the AZ.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23895_You_run_detectors_across_three_AZs_behind_a_load_balancer_bu.md>)
- [Deep] [Explain why cross-AZ or cross-region data transfer can quietly cost more than compute in a data platform. What design decisions (data locality replication topology reader placement) create the bill and how do you get AZ-affinity without losing resilience?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24314_Explain_why_cross-AZ_or_cross-region_data_transfer_can_quiet.md>)
- [Deep] [Explain the trade-off between AZ-affinity routing (cheaper, lower latency) and forced cross-AZ spread (resilience). As principal, how do you get most of the cost/latency benefit while guaranteeing you don't concentrate a service into one AZ that can then take it fully down?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23583_Explain_the_trade-off_between_AZ-affinity_routing_cheaper_lo.md>)
- [Expert] [Cross-AZ data transfer is silently costing more than compute in your architecture. Explain the design decisions that create this bill (chatty services split across AZs, replication topology, misplaced caches), and redesign for AZ-affinity without sacrificing the AZ-failure resilience you built it for.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23344_Cross-AZ_data_transfer_is_silently_costing_more_than_compute.md>)

## Related concepts

- [Availability Zone Resilience](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/_index.md>) *(Cloud Fundamentals)*
- [Backpressure and Load Shedding](<../../../System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Sensitivity Points](<../../../Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>) *(Trade off Analysis)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Sample template to create a glue database\ › Availability Zone (AZ) failure recovery

