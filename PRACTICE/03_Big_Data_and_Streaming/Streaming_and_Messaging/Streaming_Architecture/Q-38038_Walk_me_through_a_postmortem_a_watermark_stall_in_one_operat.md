---
type: "practice"
difficulty: "Expert"
---

# Walk me through a postmortem: a watermark stall in one operator silently held back windows across a shared cluster and caused hours of delayed alerts before anyone noticed. What's the root cause class, and what standing controls prevent this class org-wide?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Event-time Watermarks](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a postmortem analysis of a watermark stall in a real-time streaming architecture, we are dealing with a critical issue that led to hours of delayed alerts due to an unobserved failure in one operator silently holding back windows across the shared cluster. The root cause can be classified as a coordination failure and miscommunication between operators. To prevent such issues organization-wide, several standing controls need to be implemented.

Firstly, let's understand what happened:

One operator became stalled because it was unable to advance its watermark due to some processing issue.
This stall propagated across the cluster, causing windows to lag behind.
The delayed alerts and lack of visibility meant that no one noticed the issue until much later.

Now, let's break down the root causes and prevention strategies step-by-step:

Root Cause Analysis
Watermark Mechanism: Watermarks are used in stream processing to ensure that all data up to a certain point has been processed before committing results.
Operator Coordination: Operators must coordinate their watermarks to maintain consistent state across the cluster. When one operator stalls, it can lead to cascading failures if not properly managed.
Why It Happened
Lack of Monitoring and Alerts: There were no effective mechanisms in place to detect and alert on watermark stall conditions.
Operator Independence vs. Coordination: Operators are designed to be independent but must coordinate their watermarks, which can introduce complex failure modes.
Insufficient Retries and Backoff Mechanisms: The system lacked proper retry and backoff strategies for operators that fell behind.
Prevention Strategies
Distributed Coordination Service
Implement a distributed coordination service (e.g., Kafka ZK, Etcd) to manage watermarks and ensure all operators are synchronized.
Real-time Monitoring and Alerts
Set up real-time monitoring and alerts for watermark stalls using tools like Prometheus or Grafana.
Operator Health Checks
Regular health checks of operators to detect and address issues before they propagate.
Retry and Backoff Mechanisms
Introduce retry and backoff logic within operators to handle transient failures gracefully.

#### Conclusion

The root cause is fundamentally a coordination failure in the watermark mechanism across the cluster. Effective prevention requires robust monitoring, real-time alerts, and well-designed operator coordination and health check mechanisms. Without these controls, even small issues can cascade into major delays that go unnoticed for extended periods.

#### Common Mistakes

Failing to implement distributed coordination services.
Not setting up comprehensive monitoring and alerting systems.
Underestimating the importance of retry and backoff strategies in operators.

**Comparison:**

```
Operator	Watermark (Current)	Watermark (Last Known Good)	Alert Triggered
Operator1	980	950	No
Operator2	1000	960	Yes
```

**Code:**

```python
# Example health check function for an operator
def operator_health_check():
    # Check if watermark is progressing as expected
    current_watermark = get_current_watermark()
    last_known_good_watermark = get_last_known_good_watermark()

    if current_watermark - last_known_good_watermark > THRESHOLD:
        log.error("Watermark stall detected")
        return False

    return True
```

**Example input / output:**

```

Input:

current_watermark = 1000
last_known_good_watermark = 950
THRESHOLD = 50

Output:

PLAINTEXT
Copy
Watermark stall detected, triggering health check and alert.
```

**Data:**

```json
{
  "watermark": {
    "current": 1000,
    "last_known_good": 950,
    "threshold": 50
  }
}
```

**Flow:**

1. Yes
2. No
3. Operator Health Check
4. Is Watermark Stalled?
5. Log Error and Alert
6. Continue Processing

