---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Rate_Limiting"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["Redis"]
---

# Distributed Rate Limiting

**L4 Advanced** · [↑ Distributed Systems Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [You need per-user rate limiting on an LLM API across 20 gateway pods with a hard tokens-per-minute quota. Compare a shared Redis token bucket against local buckets with periodic reconciliation, and what consistency you sacrifice with each under a Redis blip.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23148_You_need_per-user_rate_limiting_on_an_LLM_API_across_20_gate.md>)
- [Intermediate] [You must enforce a per-tenant ingest quota (events/sec) across 20 collector gateways with a hard global cap. Compare a shared Redis token bucket against local buckets with periodic reconciliation, and what consistency you sacrifice under a Redis blip.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23869_You_must_enforce_a_per-tenant_ingest_quota_events_sec_across.md>)
- [Deep] [You must enforce a hard global tokens-per-minute quota for an LLM tenant across regional gateways while tolerating a region partition. Explain why perfect global enforcement and partition tolerance conflict and where you'd land between over- and under-admitting.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23829_You_must_enforce_a_hard_global_tokens-per-minute_quota_for_a.md>)
- [Deep] [You need distributed rate limiting on writes to protect a shared database across 50 API nodes with a hard global quota. Compare a shared-Redis token bucket versus local buckets with periodic reconciliation, and what consistency you sacrifice.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24133_You_need_distributed_rate_limiting_on_writes_to_protect_a_sh.md>)
- [Deep] [You need distributed rate limiting across 50 API gateway nodes with a global limit. Compare token bucket in a shared Redis vs local buckets with periodic reconciliation. What consistency do you sacrifice?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23265_You_need_distributed_rate_limiting_across_50_API_gateway_nod.md>)
- [Deep] [Explain how you'd design a distributed rate limiter enforced at 50 API gateway nodes using a shared token store versus local buckets with reconciliation. Which do you pick for a hard billing quota and why?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23452_Explain_how_you_d_design_a_distributed_rate_limiter_enforced.md>)
- [Deep] [Explain how you'd design a distributed rate limiter with a hard org-wide quota that must tolerate a region partition. Why do perfect global limiting and partition tolerance conflict and where do you land pragmatically?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23508_Explain_how_you_d_design_a_distributed_rate_limiter_with_a_h.md>)
- [Deep] [Explain how you'd rate-limit an LLM API per customer across a fleet of gateway nodes with a hard global token-per-minute quota. Compare a shared Redis token bucket versus local buckets with reconciliation, and what consistency you sacrifice.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23667_Explain_how_you_d_rate-limit_an_LLM_API_per_customer_across_.md>)

## Related concepts

- [Distributed Rate Limiter Design](<../../System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>) *(High Level Design)*
- [Rate Limiting at the Gateway](<../../APIs/API_Gateway/Rate_Limiting_at_the_Gateway.md>) *(API Gateway)*
- [API Rate Limiting](<../../APIs/REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*

