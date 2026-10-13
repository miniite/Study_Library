---
id: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Testing_in_Production"
level: 5
type: "deep-dive"
topic: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals"
tools: []
---

# Testing in Production

**L5 Expert** · [↑ Cloud Fundamentals](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is a feature flag and how could it help you roll out a new LLM model to only some users first?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08404_What_is_a_feature_flag_and_how_could_it_help_you_roll_out_a_.md>)
- [Deep] [What is graceful degradation and how would your AI feature keep working if the vector database is temporarily down?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08456_What_is_graceful_degradation_and_how_would_your_AI_feature_k.md>)
- [Deep] [How do you design a circuit breaker so it opens on real failures but does not trip on a brief latency spike, and how do you tune half-open recovery?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08626_How_do_you_design_a_circuit_breaker_so_it_opens_on_real_fail.md>)
- [Deep] [You're setting the org's stance on chaos engineering for the ML platform: injecting GPU-node failures, spot reclamations, and feature-store outages in production. How do you make it a norm and convince risk-averse leaders it's safe?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08327_You_re_setting_the_org_s_stance_on_chaos_engineering_for_the.md>)
- [Deep] [How would you architect the org's approach to safely testing model changes in production at scale (shadow traffic, mirrored inference, dark launches) as a governed capability, so teams validate under real load without affecting user-facing predictions?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08353_How_would_you_architect_the_org_s_approach_to_safely_testing.md>)
- [Deep] [How would you architect the org's approach to safely testing in production at scale (shadow traffic, dark launches, mirroring) as a governed capability, so teams validate under real load without risking customers?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08953_How_would_you_architect_the_org_s_approach_to_safely_testing.md>)
- [Expert] [How would you design a system to safely test in production, using shadow traffic or dark launches, before turning a feature on for real users?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08600_How_would_you_design_a_system_to_safely_test_in_production_u.md>)
- [Expert] [Describe how Conway's law has shaped an ML platform architecture you saw go wrong, and how you would deliberately reorganize teams to get the model, feature, and accelerator-ownership boundaries you actually want.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08338_Describe_how_Conway_s_law_has_shaped_an_ML_platform_architec.md>)

## Related concepts

- [Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Backpressure and Load Shedding](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Deployment Gating Trade-offs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Deployment_Gating_Trade-offs.md>) *(CI CD)*

## References

- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Overview of Principles
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Capital One Adoption and Evolution of Chaos Engineering
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Creating Foresight
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Humanistic Chaos
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Open Minds, Open Science, and Open Chaos
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Chaos Engineering on a Database
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Conclusion

