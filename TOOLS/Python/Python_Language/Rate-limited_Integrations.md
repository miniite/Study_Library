---
id: "TOOLS/Python/Python_Language/Rate-limited_Integrations"
level: 4
type: "concept"
topic: "TOOLS/Python/Python_Language"
tools: ["Python", "Redis"]
---

# Rate-limited Integrations

**L4 Advanced** · [↑ Python Language](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../_index.md>)
- [Redis](<../../Redis/_index.md>)

## Practice questions

- [Intermediate] [How would you handle rate limits and transient failures when pulling data from a third-party API in a Python extraction job?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37881_How_would_you_handle_rate_limits_and_transient_failures_when.md>)
- [Intermediate] [How would you write a Python script that queries the GitHub API to enforce a policy across hundreds of repos (branch protection, required checks) and handles secondary rate limits gracefully?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37060_How_would_you_write_a_Python_script_that_queries_the_GitHub_.md>)
- [Intermediate] [Why do LLM APIs return rate-limit (429) errors, and how would you add a simple retry with a short wait in your Python client?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37393_Why_do_LLM_APIs_return_rate-limit_429_errors_and_how_would_y.md>)
- [Deep] [How would you implement a token-bucket rate limiter in Python backed by Redis so it works correctly across multiple app instances, and why is a naive in-memory limiter wrong?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36766_How_would_you_implement_a_token-bucket_rate_limiter_in_Pytho.md>)
- [Deep] [How would you implement rate limiting per API key in a Python gateway using a sliding window or token bucket, and where do you store counters for correctness across replicas?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36887_How_would_you_implement_rate_limiting_per_API_key_in_a_Pytho.md>)
- [Deep] [How would you design a rate-limited outbound integration in Python where the third party allows 10 req/s, coordinating the limit across all your worker instances?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36915_How_would_you_design_a_rate-limited_outbound_integration_in_.md>)
- [Deep] [boto3 throttles you with ClientError ThrottlingException under a large batch operation. How would you implement adaptive retry and back off correctly, and what does botocore's built-in retry config already give you?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37024_boto3_throttles_you_with_ClientError_ThrottlingException_und.md>)
- [Deep] [How would you implement a Python rate limiter on outbound remediation actions so that a bug that fires 1000 triggers can't execute 1000 destructive actions against your infra?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37810_How_would_you_implement_a_Python_rate_limiter_on_outbound_re.md>)

## Related concepts

- [API Rate Limiting](<../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*
- [Rate Limiting at the Gateway](<../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Rate_Limiting_at_the_Gateway.md>) *(API Gateway)*
- [Distributed Rate Limiter Design](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>) *(High Level Design)*

