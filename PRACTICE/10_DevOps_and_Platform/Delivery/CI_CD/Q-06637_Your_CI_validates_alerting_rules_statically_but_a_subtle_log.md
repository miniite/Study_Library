---
type: "practice"
difficulty: "Deep"
---

# Your CI validates alerting rules statically but a subtle logic bug still shipped; how would you add a replay-based dynamic test against recorded metrics to catch what static linting misses?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Flaky Tests](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Flaky_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we face a critical challenge where our CI/CD validation process catches most issues but fails to detect subtle logic bugs in alerting rules. To address this, I propose adding a dynamic replay test that uses recorded metrics from past production runs to simulate the environment and catch these elusive bugs.

#### Triage
What’s Being Asked

The question is asking how to design a CI/CD validation step that dynamically replays recorded metrics against our alerting rules to identify logic issues that static linting might miss. The key elements are:

Static Validation: Ensures basic syntax and structure of the alerting rules.
Dynamic Testing: Uses historical production data to simulate real-world scenarios and validate rule behavior.

#### Assumptions
We have access to historical metrics from past production runs.
Our CI/CD pipeline can handle replay tests without significant overhead or downtime.
The alerting system is stateful, meaning rules might depend on historical context.

#### First Principles Solution
Step 1: Define the Replay Environment

First, we need a consistent and isolated environment to replay recorded metrics:

Environment Setup: Ensure the test environment mirrors production as closely as possible (e.g., same database, infrastructure, and dependencies).
Metric Collection: Record key metrics from past production runs that are relevant to our alerting rules.
Step 2: Develop Test Cases

Develop comprehensive test cases based on historical data:

Test Case Generation: Create scenarios that cover various edge cases and typical use cases.
Rule Coverage: Ensure all alerting rules are tested, including those that might be stateful or context-dependent.
Step 3: Implement the Replay Logic

Implement a mechanism to replay recorded metrics in the test environment:

Replay Mechanism: Develop scripts or tools that can play back these metrics at specified intervals.
Rule Evaluation: Evaluate alerting rules during each time window of the replayed metrics to identify any issues.
Step 4: Integrate with CI/CD Pipeline

Integrate the replay logic into our existing CI/CD pipeline:

Pipeline Modification: Add a new step in the pipeline for dynamic testing.
Failure Handling: Ensure that failed tests can be automatically identified and reported back through the pipeline.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To effectively add a replay-based dynamic test, we must carefully design and implement a robust testing strategy. Key considerations include:

Ensuring historical data is representative of real-world scenarios.
Handling stateful rules that depend on past events.
Avoiding false negatives or positives by thoroughly validating the test cases.

#### Common mistakes
to avoid:

Ignoring edge cases in test case generation.
Failing to properly isolate the testing environment from production.
Not addressing the complexity of stateful alerting rules.

#### Follow-up
questions might include:

How would you ensure that the replayed data accurately reflects production?
What strategies could be used to handle different types of statefulness in alerting rules?

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Replay-based Dynamic Testing Timeline
    section Recording Metrics
        2023-10-01T00:00:00Z,60h,Collect and store metrics from production
    section Test Environment Setup
        after recording Metrics,48h,Setup test environment to mimic production
    section Replay & Evaluate Rules
        after Test Environment Setup,24h,Replay recorded metrics and evaluate rules
    section Reporting Results
        after Replay & Evaluate Rules,12h,Analyze results and report any failures
```

**Code:**

```python
# Example Python script for dynamic replay testing

def replay_metrics(metrics, interval):
    start_time = metrics['start_time']
    end_time = metrics['end_time']

    while start_time < end_time:
        current_time = start_time + interval
        # Simulate metric collection and alerting rule evaluation at each time step
        for metric in collect_metrics(current_time):
            evaluate_rule(metric)

        start_time += interval

def collect_metrics(time_step):
    # Placeholder function to simulate metric collection
    return [metric for metric in recorded_data if metric['timestamp'] == time_step]

def evaluate_rule(metric):
    # Placeholder function to evaluate alerting rules against collected metrics
    pass
```

**Example input / output:**

```text
Input:
{
  "start_time": "2023-10-01T00:00:00Z",
  "end_time": "2023-10-05T23:59:59Z",
  "interval": 60,
  "recorded_data": [
    {"timestamp": "2023-10-01T00:00:00Z", "value": 100},
    {"timestamp": "2023-10-02T00:00:00Z", "value": 150}
  ]
}

Output:
Evaluation Results:
- Rule A passed at 2023-10-01
- Rule B failed at 2023-10-02 (expected value was not met)
```

**Data:**

```json
{
  "start_time": "2023-10-01T00:00:00Z",
  "end_time": "2023-10-05T23:59:59Z",
  "interval": 60,
  "recorded_data": [
    {"timestamp": "2023-10-01T00:00:00Z", "value": 100},
    {"timestamp": "2023-10-02T00:00:00Z", "value": 150}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Record Metrics
4. Is Test Environment Ready?
5. Setup Test Environment
6. Replay Recorded Metrics
7. Evaluate Alerting Rules
8. Report Results
9. Prepare and Deploy TestEnvironment

