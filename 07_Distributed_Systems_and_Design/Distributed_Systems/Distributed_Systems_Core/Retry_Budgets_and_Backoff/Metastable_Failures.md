---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Metastable_Failures"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Metastable Failures

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [You capture prompt/response pairs to a log pipeline for future fine-tuning, and during a traffic spike the logger drops records. Explain where backpressure should live so logging never takes down generation but you don't silently lose the training data.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23155_You_capture_prompt_response_pairs_to_a_log_pipeline_for_futu.md>)
- [Deep] [Explain what happens to in-flight requests when Kubernetes drains a node during a cluster autoscaler scale-down and how PodDisruptionBudgets preReadiness and connection draining together prevent dropped traffic.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23424_Explain_what_happens_to_in-flight_requests_when_Kubernetes_d.md>)
- [Deep] [Your auto-scaling remediation reacts to a CPU alert, but the workload is actually latency-bound behind a slow dependency, so scaling out did nothing and made recovery slower. Explain why the scaling signal is wrong and what metric you'd trigger remediation on instead.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23873_Your_auto-scaling_remediation_reacts_to_a_CPU_alert_but_the_.md>)
- [Deep] [You detect and want to auto-scale a service on queue depth, but the autoscaler and a downstream failing dependency created a death spiral where scaling out made recovery impossible. Reconstruct the feedback loop and the guardrails (cooldown, dependency-aware scaling) you'd add.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23935_You_detect_and_want_to_auto-scale_a_service_on_queue_depth_b.md>)
- [Deep] [Explain how you'd detect and remediate an unbounded queue in the telemetry pipeline that quietly absorbed a slow consumer for hours before filling and failing all at once. What age-based and depth-based alerts surface this gradually, and why is an unbounded shock-absorber queue an anti-pattern here?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24043_Explain_how_you_d_detect_and_remediate_an_unbounded_queue_in.md>)
- [Expert] [Explain the concept of 'control-loop interference' when multiple autonomous controllers (autoscaling, rebalancing, retry, circuit-breaking) act on the same system. As a principal, how do you architect these to avoid oscillation and to have a single coordinating authority under stress rather than fighting controllers?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23387_Explain_the_concept_of_control-loop_interference_when_multip.md>)
- [Expert] [Explain why an AIOps anomaly-detection pipeline that consumes the same telemetry it's meant to protect can enter a blind spot precisely when the observability backend degrades. How would you architect the detector's own health signal to be independent of the pipeline it monitors?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23949_Explain_why_an_AIOps_anomaly-detection_pipeline_that_consume.md>)
- [Expert] [Explain how you'd design a kill-switch that instantly disables a misbehaving auto-remediation subsystem during an incident, and why the kill-switch must not depend on the observability or control plane it might need to disable. What's your break-glass path when the AIOps automation is itself the problem?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23985_Explain_how_you_d_design_a_kill-switch_that_instantly_disabl.md>)

## Related concepts

- [Sensitivity Points](<../../../Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>) *(Trade off Analysis)*
- [Backpressure and Load Shedding](<../../../System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Correlated Failure Prevention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>) *(Cloud Fundamentals)*

## References

- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Overview of Principles › Minimize Blast Radius
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Slack's Disasterpiece Theater › The Process
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Slack's Disasterpiece Theater › The Exercise
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Slack's Disasterpiece Theater › Conclusion
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Cautions and limits
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Incident response

