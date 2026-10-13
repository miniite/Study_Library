---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Retry Budgets and Backoff

**L4 Advanced** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Metastable Failures](<Metastable_Failures.md>)
- [Fail-open vs Fail-closed](<Fail-open_vs_Fail-closed.md>)

## Practice questions

- [Intermediate] [Your LLM gateway retries on 5xx from the model backend, and during a GPU outage those retries form a storm that keeps the backend from recovering. Explain the mechanism and how exponential backoff with jitter, a retry budget, and a circuit breaker break it.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23152_Your_LLM_gateway_retries_on_5xx_from_the_model_backend_and_d.md>)
- [Intermediate] [You add retries around a remediation call to a flaky infra API, but a retry after an ambiguous timeout scaled a group twice. Explain which errors are safely retryable, why timeouts on the scale call are ambiguous, and how you'd make the ambiguous ones safe.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23899_You_add_retries_around_a_remediation_call_to_a_flaky_infra_A.md>)
- [Deep] [Your service retries a failed write during a database brownout and the retry storm keeps the DB from recovering. Explain the mechanism and the fixes: exponential backoff with jitter, a retry budget, and a circuit breaker around the query.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24136_Your_service_retries_a_failed_write_during_a_database_browno.md>)
- [Deep] [Your service retries on failure and during a downstream outage the retries create a retry storm that keeps the downstream from recovering. Explain the mechanism and the fixes: exponential backoff, jitter, retry budgets, circuit breakers.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23260_Your_service_retries_on_failure_and_during_a_downstream_outa.md>)
- [Deep] [Explain the circuit breaker pattern: closed / open / half-open states. How do you pick the failure threshold and the half-open probe rate so you neither trip too early nor hammer a recovering service?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23262_Explain_the_circuit_breaker_pattern_closed_open_half-open_st.md>)
- [Deep] [Explain why 'retry on 5xx but not 4xx' is too simplistic. Which errors are safely retryable, which are ambiguous (timeouts on non-idempotent writes), and how do you make ambiguous ones safe?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23304_Explain_why_retry_on_5xx_but_not_4xx_is_too_simplistic_Which.md>)
- [Deep] [You add retries to your model-serving client for transient errors and during a GPU outage the retries create a storm that prevents recovery. Explain how you'd add exponential backoff with jitter, a retry budget, and a circuit breaker around the inference call.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23666_You_add_retries_to_your_model-serving_client_for_transient_e.md>)
- [Deep] [Your pipeline retries a failed load against a downstream warehouse and the retries create a storm that keeps it from recovering. Explain exponential backoff jitter a retry budget and a circuit breaker in the loader and why the budget is the key constraint.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24336_Your_pipeline_retries_a_failed_load_against_a_downstream_war.md>)

## Related concepts

- [Idempotent Order APIs](<../../../System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*
- [Exactly Once Scheduled Jobs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Exactly_Once_Scheduled_Jobs.md>) *(Cloud Fundamentals)*
- [Long-running Operations](<../../../APIs/REST_APIs/Long-running_Operations.md>) *(REST APIs)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Retry Logic in SSIS
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Automated Recovery and Retry Strategies
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Expert notes
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Note
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Encountering Complex Systems › Example 2: Customer-Induced Retry Storm
- *Think Distributed Systems* — Dominik Tornow — Failure tolerance › 3.3 In practice
- *Ultimate Data Engineering with Databricks* — Mayank Malhotra — Chapter 7: Data Orchestration and Workflow Management › Techniques for Handling Workflow Failures and Retries

