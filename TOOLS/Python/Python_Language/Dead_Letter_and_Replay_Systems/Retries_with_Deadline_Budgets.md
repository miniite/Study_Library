---
id: "TOOLS/Python/Python_Language/Retries_with_Deadline_Budgets"
level: 5
type: "deep-dive"
topic: "TOOLS/Python/Python_Language"
tools: ["Python", "AWS", "Redis"]
---

# Retries with Deadline Budgets

**L5 Expert** · [↑ Python Language](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../_index.md>)
- [AWS](<../../../AWS/_index.md>)
- [Redis](<../../../Redis/_index.md>)

## Practice questions

- [Basic] [How would you time how long an LLM API call takes in Python, and why do you care about latency in a user-facing GenAI app?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37406_How_would_you_time_how_long_an_LLM_API_call_takes_in_Python_.md>)
- [Intermediate] [Why is it important to set a request timeout on LLM API calls in Python, and what happens to your app if you don't?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37451_Why_is_it_important_to_set_a_request_timeout_on_LLM_API_call.md>)
- [Deep] [How would you design a retry decorator that's async-aware, respects a deadline budget across attempts, and doesn't retry non-idempotent or 4xx errors?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36789_How_would_you_design_a_retry_decorator_that_s_async-aware_re.md>)
- [Deep] [How would you design a Python retry-and-fallback chain across multiple LLM providers so an outage or a 500 from one provider transparently fails over to another without duplicating the request?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37497_How_would_you_design_a_Python_retry-and-fallback_chain_acros.md>)
- [Deep] [How would you set up a Python canary rollout for a new prompt or model version in a GenAI service, comparing live faithfulness and latency before promoting to 100%?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37528_How_would_you_set_up_a_Python_canary_rollout_for_a_new_promp.md>)
- [Deep] [How would you design graceful degradation in a Python GenAI service: when the primary model or vector store is down, what does the fallback return, and how do you make the degraded mode observable and bounded rather than silently wrong?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37552_How_would_you_design_graceful_degradation_in_a_Python_GenAI_.md>)
- [Deep] [You must migrate a large sync Python (Flask/WSGI) GenAI service to async without a big-bang rewrite. How would you sequence it so both worlds interoperate safely and you can measure the latency/throughput win at each step?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37574_You_must_migrate_a_large_sync_Python_Flask_WSGI_GenAI_servic.md>)
- [Deep] [How would you architect the retry strategy for LLM calls in Python so it respects a per-request deadline budget across attempts, backs off with jitter, and never retries a non-idempotent tool call or a content-policy refusal?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37577_How_would_you_architect_the_retry_strategy_for_LLM_calls_in_.md>)

## Related concepts

- [Long-running Operations](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Long-running_Operations.md>) *(REST APIs)*
- [Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>) *(REST APIs)*
- [Fan-out Trade-offs](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*

