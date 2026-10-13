---
id: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design"
tools: []
---

# Compensating Transactions

**L4 Advanced** · [↑ Low Level Design](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Lock Granularity Design](<Lock_Granularity_Design.md>)
- [Two-level Cache Design](<Two-level_Cache_Design.md>)

## Practice questions

- [Intermediate] [Design a retry-with-backoff utility class. What are its knobs and how do you cap total attempts?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31531_Design_a_retry-with-backoff_utility_class_What_are_its_knobs.md>)
- [Intermediate] [Design a class model for a multi-step import flow (upload CSV, validate, preview, commit) where each step's output feeds the next and users can abandon midway; how do you model the pipeline state and cleanup?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31766_Design_a_class_model_for_a_multi-step_import_flow_upload_CSV.md>)
- [Deep] [Design a Circuit Breaker class. What are its states and what triggers each transition?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31532_Design_a_Circuit_Breaker_class_What_are_its_states_and_what_.md>)
- [Deep] [Design a loyalty/reward-points engine where earning rules change frequently (double points on weekends) without a deploy; how do you model rules as data and keep redemption idempotent?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31397_Design_a_loyalty_reward-points_engine_where_earning_rules_ch.md>)
- [Deep] [A failed recurring charge triggers dunning; how would you design the retry-and-notify state machine so it's observable, doesn't double-charge, and can be paused for a customer in a dispute?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31630_A_failed_recurring_charge_triggers_dunning_how_would_you_des.md>)
- [Deep] [You must design a poison-pill-safe consumer that can quarantine a bad message but keep processing later ones without losing at-least-once guarantees; how do you model offset commit vs quarantine so nothing is lost or skipped?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31755_You_must_design_a_poison-pill-safe_consumer_that_can_quarant.md>)
- [Deep] [A principal loop: design 'a class model where an operation must be both undoable and auditable' so the undo and the audit entry are derived from the same recorded intent; how do you avoid the classic bug where undo works but the audit log shows only the forward action?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31985_A_principal_loop_design_a_class_model_where_an_operation_mus.md>)
- [Expert] [A Circuit Breaker you added is flapping in prod, opening and closing every few seconds and making things worse; how do you debug the state transitions and tune the thresholds and half-open probe?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31619_A_Circuit_Breaker_you_added_is_flapping_in_prod_opening_and_.md>)

## Related concepts

- [Idempotent Order APIs](<../../System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*
- [Checkout System Design](<../../High_Level_Design/Checkout_System_Design.md>) *(High Level Design)*
- [Exactly Once Dedup Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>) *(Data Structures and Algorithms)*

