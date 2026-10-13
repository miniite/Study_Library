---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Load_Balancing_Stateful_Inference"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Load Balancing Stateful Inference

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [You join delayed human feedback (a thumbs-up hours later) back to the original LLM generation across a high-throughput fleet. Explain the join-key, storage, and windowing design and how you handle feedback that arrives after your retention window.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23187_You_join_delayed_human_feedback_a_thumbs-up_hours_later_back.md>)
- [Intermediate] [A user abandons a long streaming generation but your GPU keeps producing tokens no one reads, wasting capacity. Explain how you'd propagate client-disconnect cancellation from the gateway through the router to the model worker to free the KV slot promptly.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23823_A_user_abandons_a_long_streaming_generation_but_your_GPU_kee.md>)
- [Deep] [An LLM inference tier hit a metastable overload: a spike filled the queue, clients timed out and retried, and the queue never drained even after traffic dropped. Explain the sustaining feedback loop specific to token generation and how admission control plus request cancellation recover it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23156_An_LLM_inference_tier_hit_a_metastable_overload_a_spike_fill.md>)
- [Deep] [A single tenant on your multi-tenant LLM platform sends a burst of very long-context requests that consume most of the KV-cache, starving other tenants. Explain how you'd detect this noisy neighbor and design fair sharing that accounts for token cost, not request count.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23817_A_single_tenant_on_your_multi-tenant_LLM_platform_sends_a_bu.md>)
- [Deep] [You run three inference replicas behind a load balancer and round-robin routing causes uneven KV-cache memory because sessions have different context lengths. Explain why round-robin is wrong for LLM serving and what a KV-cache-aware or least-loaded router does instead.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23846_You_run_three_inference_replicas_behind_a_load_balancer_and_.md>)
- [Deep] [Explain why client-disconnect propagation matters enormously for LLM serving cost: if a client abandons a request, the GPU may keep generating tokens no one will read. How would you design cancellation to propagate through the router and free the KV-cache slot promptly across the fleet?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23751_Explain_why_client-disconnect_propagation_matters_enormously.md>)
- [Expert] [An LSM-tree store's read latency degrades over time until compaction runs, then spikes CPU/IO. Explain write amplification, read amplification, and space amplification, and how you'd tune compaction.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23272_An_LSM-tree_store_s_read_latency_degrades_over_time_until_co.md>)
- [Expert] [You run replicas of a model server behind a load balancer and see uneven GPU memory usage because of KV-cache. Explain why round-robin load balancing is wrong for LLM serving and what a KV-cache-aware or least-loaded router does instead.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23627_You_run_replicas_of_a_model_server_behind_a_load_balancer_an.md>)

## Related concepts

- [Load Shedding Responses](<../../../APIs/REST_APIs/Load_Shedding_Responses.md>) *(REST APIs)*
- [Fan-out Trade-offs](<../../../System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Distributed Cache Design](<../../../System_Design/High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — J › L

