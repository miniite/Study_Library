---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Exactly_Once_via_Idempotency"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns"
tools: []
---

# Exactly Once via Idempotency

**L5 Expert** · [↑ Concurrency Patterns](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [Two service instances both run a "read balance, check, decrement" flow on the same row and users occasionally overdraft. Walk me through exactly where the race is and three different ways to close it (DB-level, app-level, and design-level).](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09835_Two_service_instances_both_run_a_read_balance_check_decremen.md>)
- [Deep] [Two requests create the same "get-or-create" resource concurrently and you end up with duplicate rows despite a unique constraint being intended. Walk through the race and the idempotent upsert design that eliminates it.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09856_Two_requests_create_the_same_get-or-create_resource_concurre.md>)
- [Deep] [A time-of-check-to-time-of-use (TOCTOU) bug lets two requests both pass a "seat is available" check and both book it. Explain TOCTOU generally and how you'd atomically check-and-reserve in the DB.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09869_A_time-of-check-to-time-of-use_TOCTOU_bug_lets_two_requests_.md>)
- [Deep] [Two POST requests for the same idempotency key arrive simultaneously (double-click). Walk through every interleaving and design storage so the second request returns the first's response without re-executing.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09917_Two_POST_requests_for_the_same_idempotency_key_arrive_simult.md>)
- [Deep] [You're setting SLOs for a system whose correctness depends on serialized processing. How do you express "zero double-processing" as an error budget, decide the acceptable violation rate, and design the guardrails so the org doesn't trade correctness for velocity unknowingly?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09948_You_re_setting_SLOs_for_a_system_whose_correctness_depends_o.md>)
- [Expert] [A retry-with-backoff on a downstream timeout causes duplicate side effects (double charge). Explain the interaction between retries, timeouts, and idempotency, and design an idempotency-key store that's itself race-safe.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09893_A_retry-with-backoff_on_a_downstream_timeout_causes_duplicat.md>)
- [Expert] [A transactional-outbox poller and the app both write, and you get duplicate/out-of-order events. Walk through the concurrency guarantees the outbox does and doesn't give, and how you order/dedup on the consumer.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09902_A_transactional-outbox_poller_and_the_app_both_write_and_you.md>)
- [Expert] [A flash-sale postmortem shows the reservation ledger became the single serialization bottleneck and throttled the whole checkout path. Walk through how you'd re-architect to keep correctness (no oversell) while removing the global hot spot, and how you'd prove the new design under load before peak.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09951_A_flash-sale_postmortem_shows_the_reservation_ledger_became_.md>)

## Related concepts

- [Idempotent Order APIs](<../../System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*
- [Idempotent Ingestion](<../Distributed_Systems_Core/Idempotent_Ingestion/_index.md>) *(Distributed Systems Core)*
- [Optimistic Concurrency and Clocks](<../Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*

