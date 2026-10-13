---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Backpressure_and_Load_Shedding"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["AWS"]
---

# Backpressure and Load Shedding

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [How do you design for graceful degradation — when a recommendation service is down, the product page should still load; what patterns (fallbacks, circuit breakers) do you apply?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29656_How_do_you_design_for_graceful_degradation_when_a_recommenda.md>)
- [Deep] [Your metrics pipeline is dropping datapoints during traffic spikes and cardinality explosions — walk through how you'd add backpressure aggregation-at-edge and cardinality limits without losing critical alerting metrics.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29732_Your_metrics_pipeline_is_dropping_datapoints_during_traffic_.md>)
- [Deep] [Design the model-routing and fallback layer for a product that uses five different LLMs by cost and capability — how do you route per-request degrade to a cheaper model on overload and keep the abstraction stable as providers deprecate models every quarter?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29885_Design_the_model-routing_and_fallback_layer_for_a_product_th.md>)
- [Expert] [Set the SLO and error-budget policy for a platform of 50 teams — how do you pick target numbers, enforce budget-burn consequences, and prevent teams from gaming the SLOs?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29785_Set_the_SLO_and_error-budget_policy_for_a_platform_of_50_tea.md>)
- [Expert] [You must design for graceful degradation as a first-class property, not an afterthought — describe how you'd classify every dependency by criticality and encode the fallback behavior so degradation is tested, not hoped-for.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29807_You_must_design_for_graceful_degradation_as_a_first-class_pr.md>)
- [Expert] [Lead the design review for a new platform that must be 'right' for 5 years — what do you deliberately over-engineer now (that's expensive to add later) versus what do you consciously defer, and how do you defend both?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29837_Lead_the_design_review_for_a_new_platform_that_must_be_right.md>)
- [Expert] [You must architect how the org handles 'noisy neighbor' at every layer — CPU, DB connections, cache, and network — for a multi-tenant platform where one tenant's spike shouldn't hurt others; give the layered isolation design.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29847_You_must_architect_how_the_org_handles_noisy_neighbor_at_eve.md>)
- [Expert] [Architect the incident-response and blameless-postmortem system so recurring failure patterns actually get fixed at the architecture level — how do you turn dozens of postmortems into a prioritized reliability backlog leadership funds?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29876_Architect_the_incident-response_and_blameless-postmortem_sys.md>)

## Related concepts

- [Region Failure Resilience](<../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Region_Failure_Resilience.md>) *(Architecture Strategy)*
- [Correlated Failure Prevention](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>) *(Cloud Fundamentals)*
- [Org-wide Lineage and PII Tracking](<../System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications

