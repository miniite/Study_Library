---
type: "practice"
difficulty: "Expert"
---

# A principal disagreement: one architect wants every deploy to require an automated integration test against real downstream services; you argue that couples every team's velocity to every other's uptime. How do you frame and resolve the trade-off?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Deployment Gating Trade-offs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Deployment_Gating_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a CI/CD context, the core disagreement revolves around whether every deployment should require an automated integration test against real downstream services. On one hand, this approach ensures that changes do not break external dependencies, maintaining system integrity. On the other hand, it can significantly slow down deployments and potentially couple teams' velocities in a way that is not always optimal. To resolve this trade-off, we need to carefully analyze both sides of the argument, understand the underlying assumptions, and evaluate potential compromises.

Firstly, let’s break down the key components:

Integrity: Ensuring that new changes do not introduce bugs or breakages.
Velocity: Maintaining a high rate of deployment without unnecessary delays.

Automated integration tests against real downstream services (let's call this the "real service approach") offer robustness by validating interactions with actual external systems. However, they can be resource-intensive and slow down the CI/CD pipeline. Real teams may face challenges in terms of network latency, API rate limits, or even dependency on other teams' uptime.

To frame this trade-off effectively, we need to consider several factors:

Dependencies: How critical are the downstream services for our system's functionality?
Isolation: Can some tests be run against mocks or stubs to reduce external dependencies?
Impact on Velocity: What is the actual impact of running real service integration tests on deployment speed and team productivity?

Let’s reason through this step by step:

Dependency Assessment:

If the downstream services are mission-critical, it makes sense to ensure they work correctly.
For non-mission-critical services, mocks or stubs might be sufficient.

Testing Strategy:

Introduce a hybrid approach where critical paths and high-risk components can run real service tests.
Use mocks or stubs for less critical paths to speed up the pipeline.

Impact on Velocity:

Evaluate the current deployment process and identify bottlenecks.
Prioritize which services are most critical based on their impact on overall system stability.

Risk Management:

Implement fail-safes such as retries, timeouts, or fallback mechanisms in case of service failures during tests.
Monitor real-time metrics to quickly detect and address issues without impacting the entire deployment process.

Team Coordination:

Ensure clear communication between teams regarding their dependencies.
Establish a coordination mechanism where teams can alert each other about impending changes that might affect others.

By adopting this hybrid approach, we balance robustness with velocity. For example:

High-Priority Paths: Run real service tests for critical paths (e.g., billing systems).
Low-Priority Paths: Use mocks or stubs to speed up the pipeline while still ensuring basic functionality.

To illustrate, consider a typical deployment scenario where you have multiple services interacting:

PLAINTEXT
Service A -> Service B -> Real Downstream Service C

You could split this into two paths:

A -> B (real service test) -> C: For critical interactions.
A -> B (mock/stub) -> C: For non-critical interactions.

This way, you maintain the integrity of your system while optimizing deployment velocity.

**Comparison:**

```
Service Path	Test Type	Description
A -> B -> C	Real	High-priority path for critical interactions
A -> B	Mock/Stub	Low-priority path for non-critical interactions
```

**Code:**

```javascript
// Example code for a hybrid testing approach in a CI/CD pipeline

async function performRealServiceTest(serviceA, serviceB, realDownstreamServiceC) {
  try {
    // Run tests against real downstream services
    const response = await realDownstreamServiceC.checkAvailability();
    if (response.status === "OK") {
      return true;
    }
  } catch (error) {
    console.error("Real service test failed:", error);
    return false;
  }
}

async function performMockTest(serviceA, serviceB, mockDownstreamServiceC) {
  // Run tests against mocks or stubs
  const response = await mockDownstreamServiceC.checkAvailability();
  if (response.status === "OK") {
    return true;
  }
  return false;
}
```

**Example input / output:**

```text
Input:
- `performRealServiceTest(serviceA, serviceB, realDownstreamServiceC)`
Output:
- True if the test passes
- False if the test fails

Input:
- `performMockTest(serviceA, serviceB, mockDownstreamServiceC)`
Output:
- True if the mock test passes
- False if the mock test fails
```

**Data:**

```json
{
  "serviceA": "mockOrRealService",
  "serviceB": "mockOrRealService",
  "realDownstreamServiceC": "realServiceAPI",
  "mockDownstreamServiceC": "mockServiceAPI"
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A["Service A"];
4. B["Service B"];
5. R["Real Downstream Service C"];
6. M["Mock Service C"];
7. a1(A) --> b1(B);
8. b1(B) --> r1(R): Real Test;
9. b1(B) --> m1(M): Mock Test;
10. r1(R) --> success(OK);
11. m1(M) --> success(OK);

