---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Static_Stability"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Static Stability

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [A stakeholder says 'let's just make it CP so it's always correct.' What real availability costs does that impose, and how do you decide per-endpoint whether CP or AP is right for your product?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23251_A_stakeholder_says_let_s_just_make_it_CP_so_it_s_always_corr.md>)
- [Deep] [Your service mesh sidecars all failed to start after a control-plane cert rotation, bricking the data plane. Explain how you'd design cert rotation and last-known-good config so a control-plane hiccup degrades gracefully.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23433_Your_service_mesh_sidecars_all_failed_to_start_after_a_contr.md>)
- [Deep] [Explain the static-stability principle where the data plane keeps working when the control plane is down. How would you design failover service discovery and scaling to not depend on a live control plane during an outage?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23494_Explain_the_static-stability_principle_where_the_data_plane_.md>)
- [Expert] [Explain the principle of 'static stability' in AWS-style resilient design: the data plane keeps working even when the control plane is down. Design a system where failover, service discovery, and scaling don't depend on a live control plane during the very outage that broke it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23335_Explain_the_principle_of_static_stability_in_AWS-style_resil.md>)
- [Expert] [Design a graceful-degradation and dependency-tiering scheme for a fintech app where some flows (view balance) can serve stale/degraded data and others (transfer money) must fail closed. How do you encode this criticality in the architecture so degradation is deliberate rather than accidental?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23400_Design_a_graceful-degradation_and_dependency-tiering_scheme_.md>)
- [Expert] [A VP demands five-nines for a product whose availability is structurally capped by its weakest hard dependency. Show how you'd compute the achievable ceiling from the dependency graph, communicate the honest number, and propose the isolation/degradation changes that would actually raise it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23549_A_VP_demands_five-nines_for_a_product_whose_availability_is_.md>)
- [Expert] [Design graceful degradation for a fintech app where view-balance may serve stale data but transfer-money must fail closed. Explain how you encode dependency criticality into the architecture so degradation is deliberate, and how you continuously test the degraded paths in production.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23589_Design_graceful_degradation_for_a_fintech_app_where_view-bal.md>)
- [Expert] [You're the principal on a distributed database whose availability is structurally capped by a single hard dependency (a metadata service). Show how you'd compute the achievable availability ceiling from the dependency graph, communicate the honest number, and raise it via isolation and static stability.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24233_You_re_the_principal_on_a_distributed_database_whose_availab.md>)

## Related concepts

- [Backpressure and Load Shedding](<../../../System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Control Plane Failures](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Control_Plane_Failures.md>) *(Cloud Fundamentals)*
- [HA Failover Design](<../../../../01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design.md>) *(High Availability)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Distributed Systems, Scalability, and Designing for Failure
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Datastore Field Guide › Availability
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Engineering's Role › Infrastructure control
- *Ultimate AWS Data Engineering* — Rathish Mohan — Building Fault-Tolerant Architectures › Example Case Study: Self-Healing Architecture for a Travel Booking Platform
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Chapter 1. Reliable, Scalable, and Maintainable Applications
- *Think Distributed Systems* — Dominik Tornow — Thinking in distributed systems Models, mindsets, and mechanics › 1.4.2 Scalability and reliability

