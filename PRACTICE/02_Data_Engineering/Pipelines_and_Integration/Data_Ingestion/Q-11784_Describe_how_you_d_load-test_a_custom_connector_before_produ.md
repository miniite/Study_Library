---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd load-test a custom connector before production, including edge cases like pagination cursors expiring mid-run.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Managed Ingestion and Re-syncs](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Managed_Ingestion_and_Re-syncs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To load-test a custom data ingestion connector using Airbyte or Fivetran, we need to ensure that it can handle various scenarios, especially those involving pagination and cursor management. The goal is to validate its performance, stability, and reliability under realistic conditions before deploying it in production.

#### Triage

The question asks us to develop a comprehensive test plan for a custom connector. This involves understanding the core functionality of the connector, simulating real-world data volumes, testing edge cases such as cursor expiration during long-running jobs, and evaluating the overall performance and stability.

#### Assumptions
The connector has an API endpoint that supports pagination through cursors.
We have access to a test dataset with varying data volumes and complexity.
The environment is set up for both development and production testing.

#### Solution

Environment Setup:

Test Dataset: Prepare a large, representative dataset that mimics the expected production load. This could include a mix of small, medium, and large datasets to simulate different usage scenarios.
Test Environment: Set up a test environment similar to the production one, including network conditions, hardware resources, and any other factors that might affect performance.

Baseline Performance Testing:

Initial Load Test: Run an initial load test with small volumes of data to establish baseline metrics such as processing time, resource utilization, and error rates.
Steady State Load Test: Gradually increase the dataset size while monitoring key performance indicators (KPIs) like throughput, latency, and error rates.

Edge Case Testing:

Pagination Cursor Management: Simulate a scenario where pagination cursors expire mid-run by setting up a test that triggers cursor expiration during the load-test run.
PYTHON
# Example Python code to simulate cursor expiration
def simulate_cursor_expiration(cursors):
    for i, cursor in enumerate(cursors):
        if (i % 10) == 0:  # Simulate expiration every 10 iterations
            raise TimeoutError("Cursor expired")
Error Handling: Test how the connector handles various errors and retries. Ensure that it can recover from transient issues without causing data corruption or loss.

Scenario Testing:

Long-Running Jobs: Run a long-running job to test the connector's performance over extended periods.
Concurrent Access: Simulate concurrent access by running multiple instances of the connector in parallel to assess its scalability and concurrency handling.

Monitoring and Logging:

Real-Time Monitoring: Set up real-time monitoring tools like Prometheus, Grafana, or ELK Stack to track KPIs during load testing.
Logging: Enable detailed logging to capture any issues that arise during the test run. This includes logs from both the connector itself and the underlying infrastructure.

Post-Test Analysis:

Performance Metrics: Analyze performance metrics such as throughput, latency, and resource usage.
Failure Modes: Document any failures or unexpected behavior observed during testing.
Optimization Suggestions: Based on the test results, identify areas for improvement in both the connector and the ingestion pipeline.

#### Closing Summary

By following this comprehensive load-testing plan, we can ensure that the custom connector is robust, scalable, and reliable before deploying it in production. Key steps include setting up a representative dataset, simulating real-world conditions, testing edge cases like cursor expiration, running performance and scenario tests, and performing detailed post-test analysis.

#### Common Mistakes/Follow-Ups
Mistake 1: Ignoring edge cases such as cursor expiration during long-running jobs. This can lead to data loss or corruption.
Mistake 2: Failing to monitor real-time performance metrics, which could mask issues that only appear under load.

#### Follow-Up
Discussing the trade-offs between testing complexity and test coverage is essential. For example, while simulating cursor expiration adds complexity, it ensures that the connector can handle unexpected conditions.

**Comparison:**

| Scenario | Dataset Size | Processing Time (s) | Error Rate (%) |
|----------|--------------|---------------------|---------------|
| Small    | 1k rows      | 5                   | 0             |
| Medium   | 10k rows     | 20                  | 0             |
| Large    | 100k rows    | 300                 | 0             |

**Code:**

```python
# Example Python code to simulate cursor expiration
def simulate_cursor_expiration(cursors):
    for i, cursor in enumerate(cursors):
        if (i % 10) == 0:
            raise TimeoutError("Cursor expired")
```

**Example input / output:**

```text
Initial Load Test:
- Small dataset (1k rows): Processing time = 5s, Error rate = 0%
- Medium dataset (10k rows): Processing time = 20s, Error rate = 0%
- Large dataset (100k rows): Processing time = 300s, Error rate = 0%

Edge Case Test:
- Simulated cursor expiration: Connector recovered gracefully with no data loss.
```

**Data:**

```json
{
  "dataset": {
    "small": {"size": 1000, "complexity": "simple"},
    "medium": {"size": 10000, "complexity": "moderate"},
    "large": {"size": 100000, "complexity": "high"}
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Simulate Cursor Expiration?
5. Record Failure
6. End Test
7. Handle Recovery Gracefully
8. End Test

