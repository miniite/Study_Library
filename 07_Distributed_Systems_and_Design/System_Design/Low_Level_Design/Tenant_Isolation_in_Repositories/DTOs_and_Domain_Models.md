---
id: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/DTOs_and_Domain_Models"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design"
tools: []
---

# DTOs and Domain Models

**L5 Expert** · [↑ Low Level Design](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How would you debug a NullPointerException coming from your object graph?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31480_How_would_you_debug_a_NullPointerException_coming_from_your_.md>)
- [Intermediate] [Your team's DTOs and entities have drifted so much that mapping code is a bug source; how do you decide where mapping lives, and would you introduce a mapper library or hand-write it—defend the trade-off.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31622_Your_team_s_DTOs_and_entities_have_drifted_so_much_that_mapp.md>)
- [Intermediate] [Your service's DTO-to-entity mapping is hand-written and drifts; you're asked to decide between a code-gen mapper and reflection-based mapping; model both approaches and defend the trade-off in performance and safety.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31771_Your_service_s_DTO-to-entity_mapping_is_hand-written_and_dri.md>)
- [Deep] [You need to migrate a live 'notification preferences' model from a single boolean to per-channel-per-category granularity; how do you design the change to be backward-compatible and roll it out safely?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31620_You_need_to_migrate_a_live_notification_preferences_model_fr.md>)
- [Deep] [A principal asks you to justify introducing an org-wide error-taxonomy and exception-mapping class contract; what's the concrete cost of NOT having one at 300-service scale, and how do you roll it out so old services aren't forced to migrate on day one?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31801_A_principal_asks_you_to_justify_introducing_an_org-wide_erro.md>)
- [Deep] [You must decide the org's stance on ORM entities vs pure domain objects: should the DB shape drive the domain model or vice versa? Explain the long-term cost of each at scale and how you'd guide teams to keep persistence concerns from bleeding into the domain.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31833_You_must_decide_the_org_s_stance_on_ORM_entities_vs_pure_dom.md>)
- [Deep] [A principal loop asks how you'd design the class model for a 'projection' that serves multiple client shapes (GraphQL, REST, gRPC) from one domain aggregate without letting each transport's concerns leak into the domain; where does shaping live and how do you avoid three divergent DTOs?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31901_A_principal_loop_asks_how_you_d_design_the_class_model_for_a.md>)
- [Deep] [A bar-raiser asks how you'd design 'error boundaries end-to-end' so a typed backend error maps to a specific frontend UI treatment (retryable toast vs field error vs full-page) through a stable contract; how do you stop the mapping from becoming a giant switch that every screen re-writes?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31947_A_bar-raiser_asks_how_you_d_design_error_boundaries_end-to-e.md>)

## Related concepts

- [API Composition and BFF](<../../System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>) *(System Design Core)*
- [Multi-tenant Migrations](<../../System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*
- [Event Schema Evolution](<../../../Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>) *(Distributed Systems Core)*

## References

- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Best practice organization

