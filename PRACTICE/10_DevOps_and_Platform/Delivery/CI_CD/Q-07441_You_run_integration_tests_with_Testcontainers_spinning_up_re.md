---
type: "practice"
difficulty: "Deep"
---

# You run integration tests with Testcontainers spinning up real Postgres and Kafka, and CI got 3x slower and flakier; how would you decide when a real dependency is worth it versus a fake or shared instance?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Integration Testing and Test Doubles](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Integration_Testing_and_Test_Doubles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with an issue where integration tests are significantly slowing down CI, we need to carefully evaluate whether using real dependencies like Postgres and Kafka is necessary. The primary factors include test stability, test speed, and overall build reliability. This decision involves considering the trade-offs between a more realistic test environment and potential performance impacts.

Firstly, let's restate what the question is probing: You are asked to balance the benefits of running end-to-end tests with real dependencies against the downsides such as increased build times and flakiness. The interviewer wants to understand your thought process for making these trade-offs and how you ensure that your solution remains robust.

#### Triage
Real vs Fake Dependencies: Evaluate whether using a full-fledged Postgres and Kafka setup is truly necessary.
Performance Impact: Assess the extent of the slowdown (3x) and its impact on the CI pipeline.
Flakiness: Consider why tests are becoming more flaky, which could indicate instability in the real database or messaging system.

#### Assumptions
The real dependencies are currently being spun up using Testcontainers.
There is a significant performance hit (3x slowdown) and increased flakiness.
The test suite is critical for ensuring system integrity.
Step-by-Step Solution
1. Analyze Current Setup
Testcontainer Configuration: Check the configuration of Postgres and Kafka in Testcontainers to see if there are bottlenecks or inefficiencies that could be optimized.
Resource Utilization: Monitor resource usage during test runs (CPU, memory, network) to identify any performance issues.
2. Identify Flakiness Root Causes
Real vs Shared Instances: Determine whether the flakiness is due to real-world variability in Postgres and Kafka or if shared instances could mitigate some of this variability.
Isolation Issues: Assess if there are isolation problems between test runs that might be causing issues. For example, database transactions not being properly rolled back.
3. Consider Alternatives
Mocking Libraries: Explore using mocking libraries to simulate Postgres and Kafka behavior within tests. This can help reduce dependencies while maintaining a high degree of accuracy.
Shared Resources: Investigate the possibility of setting up shared instances that can be reused across multiple test runs, potentially reducing flakiness.
4. Impact Analysis
Performance Metrics: Collect performance metrics from both real and simulated environments to compare their impact on CI speed.
Risk Assessment: Evaluate the risk associated with using mocks versus real dependencies in terms of potential issues that might slip through testing.
5. Pilot Testing
Incremental Rollout: Implement changes incrementally (e.g., use mocks for some tests and real instances for others) to assess their impact.
Feedback Loops: Use continuous feedback from the team and monitoring tools to gauge the effectiveness of different approaches.

#### Conclusion

In this scenario, a combination of performance optimization techniques, isolation strategies, and possibly partial mocking can help balance the need for accurate end-to-end testing with CI efficiency. The key is to iteratively test various configurations until an optimal solution is found that minimizes both flakiness and slowdown.

#### Common Mistakes

Overlooking real-world variability: Assuming mocks will always capture edge cases.
Ignoring resource constraints: Failing to consider the impact of running full-fledged databases on CI resources.
Not collecting sufficient data: Jumping to conclusions without proper performance metrics or feedback loops.

**Comparison:**

```
Test Environment	Resource Usage	Performance Impact
Real Dependencies	High	3x slowdown
Mocked	Low	Minimal impact
```

**Code:**

```python
# Example code snippet for setting up Testcontainers with Postgres and Kafka
from testcontainers.postgresql import PostgresContainer
from testcontainers.kafka import KafkaContainer

postgres_container = PostgresContainer("postgres:latest")
kafka_container = KafkaContainer()

with postgres_container, kafka_container:
    # Setup database and Kafka clients here
    pass
```

**Example input / output:**

```text
- **Input**: Real Postgres and Kafka containers are spun up using Testcontainers.
- **Output**: CI pipeline becomes 3x slower, tests become flakier.

|Test Run | Real Dependencies? | Speed (sec) | Stability |
|---------|-------------------|------------|----------|
|1        | Yes               | 60         | Flaky     |
|2        | No                | 20         | Stable    |
```

**Data:**

```yaml
# Sample CI configuration for real dependencies
integration_tests:
  - name: "Postgres Integration"
    container: "postgres:latest"
  - name: "Kafka Integration"
    container: "kafka:latest"
```

**Flow:**

1. Yes
2. No
3. Real Dependency
4. Optimize Configurations
5. Monitor Performance
6. Use Shared Instances?
7. Run Partial Mocks
8. Continue Optimizing

