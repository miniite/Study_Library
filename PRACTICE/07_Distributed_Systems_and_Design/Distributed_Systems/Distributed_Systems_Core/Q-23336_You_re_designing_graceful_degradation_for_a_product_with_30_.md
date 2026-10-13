---
type: "practice"
difficulty: "Deep"
---

# You're designing graceful degradation for a product with 30 features backed by 30 services. Define a dependency criticality tiering, describe how each feature fails to a safe reduced-functionality mode instead of erroring, and explain how you test that the degradation paths actually work in production.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fail-open vs Fail-closed](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/Fail-open_vs_Fail-closed.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In designing graceful degradation for 30 features backed by 30 services, we will tier dependencies based on criticality to ensure that non-critical features are gracefully degraded while critical ones remain functional. This involves defining failure modes, implementing fallback strategies, and rigorously testing these strategies in production.

#### Triage

The question requires us to design a mechanism for graceful degradation across multiple features and their underlying services. The core of the problem is ensuring that when an error occurs, non-critical functionalities can be safely reduced or disabled without affecting critical operations. This involves understanding service dependencies, defining tiers based on criticality, implementing fallback mechanisms, and thoroughly testing these mechanisms in production.

#### Assumptions
We have a complex system with 30 features, each backed by its own set of services.
Each feature has different criticality levels; some are more essential than others.
Errors can occur at any point in the system due to various factors like service unavailability or resource exhaustion.
The goal is to maintain overall system stability and user experience.

#### First-principles Solution
Dependency Criticality Tiering
Identify critical vs non-critical services and features.
For each feature, determine its dependencies on other services.
Implement Graceful Degradation Mechanisms
Use circuit breakers to detect failures in non-critical services.
Fallback strategies: implement retries, fallback methods (e.g., caching), or alternative APIs.
Testing in Production
Simulate failure scenarios and ensure graceful degradation works as expected.
Monitor system behavior during real-world outages.
1. Dependency Criticality Tiering

We start by categorizing services into tiers based on their criticality:

Tier 1: Critical (high impact if unavailable)
Tier 2: Important but Less Critical
Tier 3: Non-Critical

Features are similarly categorized, with some being more critical than others. For example, features related to core business functions would be Tier 1, while less essential features might be Tier 3.

2. Implementing Graceful Degradation Mechanisms

For each feature in a non-critical tier:

Circuit Breakers: Implement circuit breakers at the service level to detect failures and automatically fallback to a pre-defined strategy.
Fallback Strategies:
Retry Logic: Retry failed requests with exponential backoff.
Cache Fallback: Use caching mechanisms to serve cached data if real-time data is unavailable.
Alternative API: Provide an alternative or simplified version of the API when primary service fails.

For critical features, ensure they always have priority and that non-critical features degrade gracefully:

Service-Level Agreements (SLAs): Define SLAs for each tier to ensure critical services are prioritized.
Failover Mechanisms: Set up failover mechanisms to automatically switch to backup systems if primary services fail.
3. Testing in Production

To test the degradation paths in production, we need a robust testing framework:

Simulate Failures: Use load testing tools (e.g., JMeter) to simulate failures and monitor system behavior.
A/B Testing: Gradually introduce failure scenarios to features and observe how users are impacted.
Real-world Scenarios: Analyze past incidents and test the system under similar conditions.

#### Closing Summary
+ Common Mistakes/Follow-ups

By tiering dependencies, implementing robust fallback strategies, and rigorously testing in production, we ensure that our distributed system can handle failures gracefully. Common mistakes include:

Overlooking critical services: Always prioritize high-impact features.
Not thoroughly testing: Ensure all failure modes are tested before deployment.
Failing to monitor: Continuous monitoring is crucial to catch issues early.

#### Follow-ups
could involve discussing specific tools and technologies used (e.g., Resilience4j, Hystrix), how to integrate with existing monitoring systems, or strategies for managing false positives in circuit breakers.

**Code:**

```
MERMAID
Copy
graph TD;
    subgraph Tier1CriticalServices
        CriticalServiceA --> CriticalServiceB
    end
    subgraph Tier2ImportantButLessCriticalServices
        ImportantServiceC --> ImportantServiceD
    end
    subgraph Tier3NonCriticalServices
        NonCriticalServiceE --> NonCriticalServiceF
    end
```

**Example input / output:**

```text
Input: Simulate a failure in NonCriticalServiceE

Output:
- CoreFunctionality feature continues to function normally.
- LessImportantFeature falls back to simplified mode (e.g., cached data).
- Monitoring shows no degradation in critical features.
```

**Data:**

```json
{
  "features": [
    {
      "name": "CoreFunctionality",
      "dependencies": ["CriticalServiceA", "ImportantServiceC"],
      "criticalityLevel": "Tier1"
    },
    {
      "name": "LessImportantFeature",
      "dependencies": ["NonCriticalServiceE"],
      "criticalityLevel": "Tier3"
    }
  ],
  "circuitBreakers": [
    {"service": "NonCriticalServiceF", "threshold": 50, "timeout": 60}
  ]
}
```

