---
id: "11_AI_and_ML/Responsible_AI/Privacy_and_PII/PII_Filters_for_LLMs"
level: 5
type: "concept"
topic: "11_AI_and_ML/Responsible_AI/Privacy_and_PII"
tools: []
---

# PII Filters for LLMs

**L5 Expert** · [↑ Privacy and PII](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [An engineer wants to log full request/response for an LLM feature to build eval sets. How do you enable that safely without hoarding PII indefinitely?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14195_An_engineer_wants_to_log_full_request_response_for_an_LLM_fe.md>)
- [Deep] [What's your strategy for privacy in prompt/response caching and semantic caching layers where cache keys or hits could leak one user's data to another?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14238_What_s_your_strategy_for_privacy_in_prompt_response_caching_.md>)
- [Deep] [Why not treat every LLM prompt as a data-transfer event requiring its own consent and residency check? Where does that stance become operationally untenable and what do you do instead?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14253_Why_not_treat_every_LLM_prompt_as_a_data-transfer_event_requ.md>)
- [Deep] [What controls do you put around an LLM's tool-calling so it can query a customer DB for a legitimate task but can't be coaxed into dumping other customers' PII?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14260_What_controls_do_you_put_around_an_LLM_s_tool-calling_so_it_.md>)
- [Deep] [What's your architecture for scoping an LLM agent's memory so long-term memory can retain useful context without becoming an unbounded, ungoverned PII repository?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14287_What_s_your_architecture_for_scoping_an_LLM_agent_s_memory_s.md>)
- [Deep] [How do you decide the retention period for LLM eval/regression datasets that must stay stable for reproducibility yet contain PII that policy says should expire?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14292_How_do_you_decide_the_retention_period_for_LLM_eval_regressi.md>)
- [Deep] [How would you design privacy telemetry for an LLM gateway that flags when responses appear to contain PII the user never provided — a possible training-leak signal?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14297_How_would_you_design_privacy_telemetry_for_an_LLM_gateway_th.md>)
- [Deep] [An interviewer follow-up: you added a PII filter before the LLM — how do you handle its false positives destroying legitimate context and its false negatives leaking data?](<../../../PRACTICE/11_AI_and_ML/Responsible_AI/Privacy_and_PII/Q-14329_An_interviewer_follow-up_you_added_a_PII_filter_before_the_L.md>)

## Related concepts

- [Governing Incident Data Reuse](<../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Incident_Data_Reuse.md>) *(Data Strategy)*
- [Compliant Access to Encrypted PII](<../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Compliant_Access_to_Encrypted_PII.md>) *(Encryption and Compliance)*
- [Sensitive Data Controls for LLMs](<../AI_Governance/Sensitive_Data_Controls_for_LLMs.md>) *(AI Governance)*

