---
type: "practice"
difficulty: "Intermediate"
---

# Your detectors process telemetry with default features when the online feature store is unavailable, but during an outage the defaults masked a real anomaly. Explain the graceful-degradation trade-off, how you'd tier the feature dependency, and how you'd test the degraded path in production.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Fail-open vs Fail-closed](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/Fail-open_vs_Fail-closed.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems like AIOps, default features are often used as a fallback when online feature stores are unavailable to ensure service availability during outages. However, using defaults can mask real anomalies, leading to false negatives and suboptimal performance. The key challenge is to strike a balance between maintaining system reliability and preserving detection accuracy. This involves tiering the feature dependencies based on criticality and testing the degraded path rigorously.

#### Triage
What's Being Asked

The question asks us to explain the trade-off involved in using default features during an outage, how to tier these dependencies, and how to test them effectively. The interviewer is probing your understanding of graceful degradation, risk management, and reliability engineering principles.

#### Assumptions
& Key Idea
Default Feature Usage: During outages, we use pre-defined or cached defaults for feature values.
Impact of Defaults: Defaults can mask real anomalies, leading to false negatives in anomaly detection.
Graceful Degradation: Ensuring the system continues to operate with acceptable performance even during partial failures.

#### First-Principles Solution

Tiering Feature Dependencies:

Critical Features: These are essential for core functionality and should always be available from the online feature store, with fallbacks only if absolutely necessary.
Non-Critical Features: These can use defaults or be derived locally without impacting critical operations.

Testing Degraded Paths:

Mock Outage Testing: Simulate outages to test how well default features handle real anomalies.
Canary Releases: Gradually introduce the degraded path in a controlled environment before rolling it out fully.

Monitoring & Alerting:

Set up monitoring for critical paths and alerts when defaults are used, so we can quickly identify issues and restore the feature store.
Detailed Steps

Identify Critical vs Non-Critical Features:

For example, if you have 10 features, label 3 as critical (e.g., CPU usage, network latency) and 7 as non-critical.

Fallback Mechanism Design:

Implement a fallback mechanism that checks the online feature store first. If unavailable, use cached or derived defaults.

#### Example
For critical features like CPU usage, always try to fetch from the online store; for non-critical features like disk I/O, use local caching or predefined values.

Testing Degraded Paths:

Mock Outages: Set up a test environment where you can simulate outages and observe how well defaults handle real anomalies.
PYTHON
# Example of simulating an outage in Python
def get_feature_value(feature_name, fallback=None):
    if online_store.is_available():
        return online_store.get(feature_name)
    else:
        print("Online store is unavailable; using default value.")
        return fallback
Canary Releases: Gradually introduce the degraded path into a production-like environment to ensure it behaves as expected.
PLAINTEXT
# Example canary release strategy
if random.random() < 0.1:  # 10% of requests
    use_default_features()
else:
    use_online_features()

Monitoring & Alerts:

Set up monitoring for critical paths to detect when defaults are being used.
PLAINTEXT
# Example alert condition
if too_many_requests_using_defaults():
    send_alert("Online feature store is unavailable; too many requests using defaults.")
Ensure alerts are triggered quickly so that the issue can be resolved before it impacts overall system performance.

#### Closing Summary
& Common Mistakes
Summary: Graceful degradation involves tiering features based on criticality, testing degraded paths rigorously, and ensuring monitoring and alerting mechanisms catch issues early.

#### Common Mistakes
Using defaults for all features without considering criticality.
Failing to test the degraded path effectively, leading to undetected anomalies during outages.
Ignoring monitoring and alerts, which can result in prolonged issues.
Final Confirmation

This solution ensures a balanced approach between maintaining system availability and preserving detection accuracy. It covers tiering dependencies, testing degradation paths, and setting up robust monitoring mechanisms.

**Comparison:**

```
Feature	Criticality	Fallback Mechanism
CPU usage	Critical	Try online store; use default if unavailable
Disk I/O	Non-Critical	Use local cache or predefined value
Network latency	Critical	Try online store; use default if unavailable
```

**Code:**

```python
def get_feature_value(feature_name, fallback=None):
    if online_store.is_available():
        return online_store.get(feature_name)
    else:
        print("Online store is unavailable; using default value.")
        return fallback

# Example of canary release strategy
if random.random() < 0.1:  # 10% of requests
    use_default_features()
else:
    use_online_features()

def too_many_requests_using_defaults():
    return online_store.get_default_usage_count() > threshold
```

**Example input / output:**

```text
Input:
- Simulate an outage in the online feature store.
Output:
- Ensure default values are used for non-critical features, but critical ones still attempt to fetch from the store.
```

**Data:**

```json
{
  "feature_dependencies": {
    "critical": ["CPU usage", "Network latency"],
    "non_critical": ["Disk I/O", "Memory usage"]
  },
  "monitoring_config": {
    "alert_thresholds": {
      "default_usage_count": 100,
      "request_frequency": 60
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Simulate Outage
4. Simulate Feature StoreUnavailability
5. Is Online Store Available?
6. Fetch from Online Store
7. Process Request Normally
8. Use Default Values forNon-Critical Features
9. Try Fetching Critical Features;Use Defaults if Unavailable
10. Process Request withDegraded Path
11. Monitor and Alert on TooMany Requests Using Defaults

