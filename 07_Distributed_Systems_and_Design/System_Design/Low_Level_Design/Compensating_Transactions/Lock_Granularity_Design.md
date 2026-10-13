---
id: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Lock_Granularity_Design"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design"
tools: ["Redis"]
---

# Lock Granularity Design

**L5 Expert** · [↑ Low Level Design](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [How does the Chain of Responsibility apply to HTTP middleware and where does next() fit?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31527_How_does_the_Chain_of_Responsibility_apply_to_HTTP_middlewar.md>)
- [Deep] [Your Chain of Responsibility middleware stack has grown to 40 handlers and ordering bugs are common; how do you make ordering explicit and testable, and how do you let one handler short-circuit safely?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31391_Your_Chain_of_Responsibility_middleware_stack_has_grown_to_4.md>)
- [Deep] [Model a request-coalescing/singleflight class so N concurrent misses for the same key trigger one downstream fetch; how do you handle the case where the in-flight fetch fails and callers must not cache the error?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31690_Model_a_request-coalescing_singleflight_class_so_N_concurren.md>)
- [Deep] [Model a class design for graceful shutdown of a service: stop accepting new work, drain in-flight requests, flush buffers, and honor a hard deadline; where do you hook the signal and how do you avoid dropping requests?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31717_Model_a_class_design_for_graceful_shutdown_of_a_service_stop.md>)
- [Deep] [You must design a class that fairly distributes a limited resource (say 100 concurrent export slots) across tenants so no tenant hogs; compare weighted fair queuing vs simple round-robin at the class level.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31763_You_must_design_a_class_that_fairly_distributes_a_limited_re.md>)
- [Deep] [A bar-raiser asks how you'd design 'a class that owns a scarce budget' (rate-limit tokens, connection slots) so borrowing and returning are symmetric and a caller that forgets to return doesn't slowly starve the pool; how do you make return automatic and detect leaks?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31981_A_bar-raiser_asks_how_you_d_design_a_class_that_owns_a_scarc.md>)
- [Expert] [How would you write a class so it is thread-safe by design rather than by adding locks everywhere?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31597_How_would_you_write_a_class_so_it_is_thread-safe_by_design_r.md>)
- [Expert] [A postmortem shows a race in a wallet-debit path that had "thread-safe" comments but a broken check-then-act; as principal how do you change the org's approach so correctness of concurrency is enforced by design (types/APIs) rather than by reviewer vigilance?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31808_A_postmortem_shows_a_race_in_a_wallet-debit_path_that_had_th.md>)

## Related concepts

- [Bulkheads and Thread Pools](<../../../Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>) *(Concurrency Patterns)*
- [Cross-service Invalidation Races](<../../../Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>) *(Concurrency Patterns)*
- [Fan-out Trade-offs](<../../High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*

