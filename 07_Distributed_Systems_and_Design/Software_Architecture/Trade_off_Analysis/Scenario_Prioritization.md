---
id: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Scenario_Prioritization"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis"
tools: []
---

# Scenario Prioritization

**L5 Expert** · [↑ Trade off Analysis](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [During Phase 1 the business owner presents goals but the metrics they cite come from a dashboard the architect says is misleading. How do you resolve a disputed baseline before you let scenario elicitation proceed?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03048_During_Phase_1_the_business_owner_presents_goals_but_the_met.md>)
- [Deep] [A tradeoff point pits a thin backend + fat client (offline capability, backend simplicity) against a thin client + fat backend (control, security, thin devices). How do you scenario-test this across security, offline-UX, and update-agility for a full-stack product?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03121_A_tradeoff_point_pits_a_thin_backend_fat_client_offline_capa.md>)
- [Deep] [How do you elicit a security scenario for your platform's software supply chain ('a malicious base image or transitive dependency reaches production')? What sensitivity points around SLSA provenance, admission policy, and signing does it expose across the pipeline?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03169_How_do_you_elicit_a_security_scenario_for_your_platform_s_so.md>)
- [Deep] [How do you weight importance versus architectural-difficulty when prioritizing platform scenarios? Give an example where a low-importance but high-difficulty scenario (say cross-cloud disaster failover) was still worth analyzing because it exposed a foundational fleet risk.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03239_How_do_you_weight_importance_versus_architectural-difficulty.md>)
- [Deep] [You find the same platform decision — mandating a shared service mesh sidecar — is a sensitivity point for latency, a risk for upgrade toil, and a tradeoff against team autonomy. How do you document one decision playing three roles without confusing stakeholders?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03262_You_find_the_same_platform_decision_mandating_a_shared_servi.md>)
- [Deep] [How do you weight importance versus architectural-difficulty when prioritizing ML platform scenarios? Give an example where a low-importance but high-difficulty scenario (say cross-region training failover) was still worth analyzing because it exposed a foundational platform risk.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03353_How_do_you_weight_importance_versus_architectural-difficulty.md>)
- [Deep] [During Phase 2 the broader on-call and product group generates scenarios that contradict the priorities the AIOps architects set in Phase 1 (on-call wants fewer pages, product wants more coverage). How do you reconcile the two prioritizations and what does the divergence tell you about org alignment?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03591_During_Phase_2_the_broader_on-call_and_product_group_generat.md>)
- [Deep] [Walk me through analyzing a modifiability scenario where the business wants to spin up a white-labeled instance for a new market in under a month. What architectural sensitivity points (multi-tenancy model, config vs code) does that scenario probe?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Q-03664_Walk_me_through_analyzing_a_modifiability_scenario_where_the.md>)

## Related concepts

- [Platform vs Product Ownership](<../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Platform_vs_Product_Ownership.md>) *(Architecture Strategy)*
- [Org-wide Lineage and PII Tracking](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Service Mesh Adoption](<../../System_Design/High_Level_Design/Service_Mesh_Adoption.md>) *(High Level Design)*

## References

- *97 Things Every Software Architect Should Know* — Richard Monson-Haefel (ed.) — Quantify
- *97 Things Every Software Architect Should Know* — Richard Monson-Haefel (ed.) — Architectural Tradeoffs
- *97 Things Every Software Architect Should Know* — Richard Monson-Haefel (ed.) — Record Your Rationale
- *97 Things Every Software Architect Should Know* — Richard Monson-Haefel (ed.) — If There Is Only One Solution, Get a Second Opinion
- *Fundamentals of Software Architecture (2nd ed.)* — Mark Richards, Neal Ford — Roadmap
- *Fundamentals of Software Architecture (2nd ed.)* — Mark Richards, Neal Ford — Extracting Architectural Characteristics from Domain Concerns
- *Fundamentals of Software Architecture (2nd ed.)* — Mark Richards, Neal Ford — The Impact of Scoping
- *Fundamentals of Software Architecture (2nd ed.)* — Mark Richards, Neal Ford — Architecture and Implementation

