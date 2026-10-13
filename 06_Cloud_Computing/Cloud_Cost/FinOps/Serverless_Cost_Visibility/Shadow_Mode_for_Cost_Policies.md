---
id: "06_Cloud_Computing/Cloud_Cost/FinOps/Shadow_Mode_for_Cost_Policies"
level: 5
type: "deep-dive"
topic: "06_Cloud_Computing/Cloud_Cost/FinOps"
tools: ["AWS", "Azure"]
---

# Shadow Mode for Cost Policies

**L5 Expert** · [↑ FinOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Azure](<../../../../TOOLS/Azure/_index.md>)

## Practice questions

- [Deep] [How would you put a proxy or gateway in front of LLM API calls so you get per-team token spend visibility and can enforce quotas before the monthly bill arrives?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26118_How_would_you_put_a_proxy_or_gateway_in_front_of_LLM_API_cal.md>)
- [Deep] [How do you architect cost governance for a feature-flag / experimentation platform where every running experiment duplicates infra paths, and stale experiments silently accrue cost forever? Design the experiment-lifecycle cost hygiene.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25976_How_do_you_architect_cost_governance_for_a_feature-flag_expe.md>)
- [Deep] [How do you architect per-workload cost attribution when your platform runs everything through a service mesh (Istio/Linkerd/Cilium) and a shared ingress/egress gateway fleet? The gateway compute and the mesh sidecars are shared cost — walk me through metering them back to originating services fairly.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26182_How_do_you_architect_per-workload_cost_attribution_when_your.md>)
- [Deep] [Explain how you'd govern the cost of a policy-as-code / admission-control layer that itself scales — thousands of OPA/Kyverno evaluations per second across the fleet add real compute. When does the governance layer's own cost and latency start to undermine the savings it enforces?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26224_Explain_how_you_d_govern_the_cost_of_a_policy-as-code_admiss.md>)
- [Deep] [Explain how you'd set a per-tenant inference cost ceiling so one customer's abusive prompt pattern (huge contexts, high call rate) can't silently erode your platform's margin. Where do you enforce the ceiling — API gateway, model router, token-budget middleware — and why?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26297_Explain_how_you_d_set_a_per-tenant_inference_cost_ceiling_so.md>)
- [Deep] [Your org lets teams pick context window size, and some default to the 1M-token window for a task needing 8K, paying for capacity they never use. How do you govern context-window right-sizing so the largest window is a deliberate, justified choice rather than a lazy default?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26410_Your_org_lets_teams_pick_context_window_size_and_some_defaul.md>)
- [Expert] [Your LLM feature's cost is dominated by prompt tokens because every request stuffs the full context. As architect, design the cost-governance strategy across prompt caching, context compression, model routing, and per-tenant budgets.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25961_Your_LLM_feature_s_cost_is_dominated_by_prompt_tokens_becaus.md>)
- [Expert] [How do you architect cost governance for platform-provided observability-as-a-service, where you resell a metrics/logs/traces pipeline to internal teams and must both cap the platform's own vendor bill and charge teams fairly for cardinality they create? Design the metering and the back-pressure.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26173_How_do_you_architect_cost_governance_for_platform-provided_o.md>)

## Related concepts

- [Database Cost Optimization](<../../../../01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*
- [Data Lifecycle and Retention](<../../../Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Data_Lifecycle_and_Retention.md>) *(Cloud Fundamentals)*
- [Multi-cloud Strategy](<../../../Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>) *(Cloud Fundamentals)*

