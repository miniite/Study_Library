---
id: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Design_Pattern_Governance"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design"
tools: []
---

# Design Pattern Governance

**L5 Expert** · [↑ Low Level Design](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is coupling and cohesion? How do you aim for low coupling and high cohesion?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31459_What_is_coupling_and_cohesion_How_do_you_aim_for_low_couplin.md>)
- [Deep] [Your DI container constructs the whole graph at startup and boot time is now 40 seconds; how do you redesign for lazy resolution while still catching wiring errors early—what's the trade-off?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31641_Your_DI_container_constructs_the_whole_graph_at_startup_and_.md>)
- [Deep] [An interviewer probes: your proposed design uses events to decouple modules; at 100x scale how do the implicit dependencies and debugging cost of event-driven LLD bite you, and where would you deliberately keep synchronous calls instead?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31810_An_interviewer_probes_your_proposed_design_uses_events_to_de.md>)
- [Deep] [An interviewer probes: you introduced a shared "utils" or "common" library everyone edits and it's now a release bottleneck across the org; what's your criterion for what becomes a shared library vs copied code, and how do you decompose the god-module?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31848_An_interviewer_probes_you_introduced_a_shared_utils_or_commo.md>)
- [Deep] [An interviewer probes your 'god reducer' on the frontend: one Redux/store slice has grown to handle 40 action types and it's the merge-conflict hotspot; how do you decompose the state model into cohesive slices with clear ownership without breaking selectors across the app?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31935_An_interviewer_probes_your_god_reducer_on_the_frontend_one_R.md>)
- [Expert] [You own the LLD standards for 40 teams shipping in 6 languages; how do you codify class-design and API conventions so they raise the floor without becoming a bureaucratic gate that senior engineers route around?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31781_You_own_the_LLD_standards_for_40_teams_shipping_in_6_languag.md>)
- [Expert] [Your org runs a mix of DDD, anemic-model, and transaction-script styles across teams and onboarding engineers are constantly confused; do you standardize on one, and if so how do you sequence a multi-year convergence given the sunk cost in each style?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31800_Your_org_runs_a_mix_of_DDD_anemic-model_and_transaction-scri.md>)
- [Expert] [You inherited a 4000-line OrderService god class that every team touches and is a merge-conflict magnet across the org; lay out a multi-quarter strangler-style decomposition into cohesive modules with no big-bang rewrite and no code freeze, and how you'd measure progress.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31847_You_inherited_a_4000-line_OrderService_god_class_that_every_.md>)

## Related concepts

- [API Composition and BFF](<../../System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>) *(System Design Core)*
- [Org-wide Lineage and PII Tracking](<../../System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Backpressure and Load Shedding](<../../High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*

## References

- *97 Things Every Software Architect Should Know* — Richard Monson-Haefel (ed.) — Fight Repetition
- *97 Things Every Software Architect Should Know* — Richard Monson-Haefel (ed.) — Prepare to Pick Two
- *Building Knowledge Graphs* — Jesús Barrasa, Maya Natarajan, Jim Webber — Dependency Knowledge Graphs
- *Claude Code: The Definitive Guide to Agentic Development* — Beyond the Getting-Started Guide
- *Claude Code: The Definitive Guide to Agentic Development* — Context Engineering
- *Claude Code: The Definitive Guide to Agentic Development* — Working with Large and Legacy Codebases
- *Claude Code: The Definitive Guide to Agentic Development* — Team Adoption Patterns
- *Fundamentals of Software Architecture (2nd ed.)* — Mark Richards, Neal Ford — Balancing Architecture and Hands-On Coding

