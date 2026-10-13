---
type: "practice"
difficulty: "Deep"
---

# You must decide the gateway's behavior when its own telemetry backend is down: does it drop metrics, buffer and risk memory, or block requests; adjudicate the failure mode, and design so losing observability never causes an availability incident yet you're not blind for long.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Resilience to Dependencies](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Resilience_to_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an API Gateway, it's crucial to balance observability (telemetry) with service availability. If the telemetry backend is down, we need a strategy that ensures we retain critical functionality without causing availability issues. The key decision points are whether to drop metrics, buffer requests and risk memory, or block requests entirely.

#### Triage
What the Question Is Really Asking

The question is probing how you balance different aspects of system design—specifically, observability vs. availability. It’s essential to understand that losing critical telemetry data can provide insights into system health but should not compromise user experience. The goal is to ensure that the API Gateway remains functional while mitigating potential risks.

#### Assumptions
Severity of Telemetry Loss: Determine if the loss of telemetry data would have a significant impact on troubleshooting and maintaining the system.
System Scale: Consider how many requests and what kind of data are involved, as this can affect memory usage.
User Experience: Evaluate whether blocking requests or buffering them could disrupt user interaction.
Step-by-Step Solution

Evaluate the Impact:

Critical vs Non-Critical Metrics: Determine which metrics are critical for troubleshooting and which can be dropped without major consequences.
Risk Assessment: Assess the risk of losing telemetry data on a short-term basis versus the risk of blocking or buffering requests, which could lead to user frustration.

Mitigation Strategies:

Buffering Requests: Buffer incoming requests in memory temporarily until telemetry is restored. This approach ensures that no requests are dropped and provides time for recovery.
Memory Management: Implement a buffer size limit to prevent excessive memory usage during the outage. Log warnings when the buffer nears capacity, indicating an impending issue.

Design the Gateway:

Fail-Safe Mechanism: Design the gateway to fail open (allow requests) and log errors locally or via another means if telemetry is unavailable.
Buffered Logging: Implement a local logging mechanism that buffers logs until it can send them to the telemetry backend. This ensures that critical information is retained, even if external services are down.

Mitigating Risk:

Telemetry Health Checks: Integrate health checks into the API Gateway itself to monitor telemetry availability. If telemetry becomes unavailable, degrade gracefully without blocking requests.
Graceful Degradation: Implement strategies that reduce non-essential operations when telemetry is down but keep essential services running.

Recovery Plan:

Automated Recovery: Set up automated recovery processes that attempt to re-establish the connection with the telemetry backend and resume normal operation once connectivity is restored.
Alerting Mechanisms: Implement alerting mechanisms to notify developers or operations teams when telemetry becomes unavailable, allowing for proactive resolution.

#### Closing Summary

In summary, the API Gateway should be designed to buffer incoming requests temporarily while retaining critical metrics in local storage. This approach ensures that user experience remains uninterrupted and critical system data is not lost. The gateway must also include health checks, recovery mechanisms, and alerting to mitigate risks and ensure quick restoration of telemetry once it becomes available again.

#### Common Mistakes/Follow-Ups
Blocking Requests: Avoid blocking requests as this can lead to high user churn and dissatisfaction.
Excessive Memory Usage: Buffering too many requests can consume significant memory resources, potentially causing the gateway itself to crash.
Ignoring Critical Metrics: Dropping critical metrics can make it difficult to diagnose issues post-failure.

**Comparison:**

```
Telemetry Backend Status	Action Taken
Available	Process request normally
Unavailable	Buffer the request and log locally
```

**Code:**

```python
# Example of a simple health check function for telemetry backend
def check_telemetry_health():
    try:
        # Perform health check with telemetry backend
        response = requests.get('http://telemetry-backend/status')
        if response.status_code == 200:
            return True
    except Exception as e:
        print(f"Telemetry health check failed: {e}")
    return False

# Example of request handling logic in API Gateway
def handle_request(request):
    buffer = []

    # Check telemetry health before processing
    if not check_telemetry_health():
        # Buffer the request temporarily and log locally
        buffer.append(request)
        print("Telemetry backend unavailable, logging request locally.")
        return "Buffering request due to telemetry unavailability."

    # Process the request normally if telemetry is available
    response = process_request(request)
    send_metrics(response)  # Send metrics to telemetry backend

    return response

# Example of sending buffered logs when telemetry becomes available again
def flush_logs():
    for log in buffer:
        print(f"Flushing log: {log}")
```

**Example input / output:**

```text
Input:
- Request to /api/v1/resource with request id req-123456789

Output when telemetry backend is down:
- Buffer the request and log locally.
Response: "Buffering request due to telemetry unavailability."

Log Output (flushed after telemetry is restored):
- Flushing log: {"timestamp": "2023-10-01T12:00:00Z", "request_id": "req-123456789", "method": "GET", "path": "/api/v1/resource"}
```

**Data:**

```json
{
  "buffer": [
    {
      "timestamp": "2023-10-01T12:00:00Z",
      "request_id": "req-123456789",
      "method": "GET",
      "path": "/api/v1/resource"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. API Gateway
4. Telemetry Health?
5. Process Request Normally
6. Buffer Request Locally
7. Flush Logs When Telemetry isRestored
8. Send Metrics to Backend (IfAvailable)

