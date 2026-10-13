---
type: "practice"
difficulty: "Expert"
---

# Design an API deprecation communication and telemetry system: how do you know exactly which integrator calls a deprecated endpoint, notify only them, and prove at sunset that traffic is truly zero before you delete the code path?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Org-wide API Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/Org-wide_API_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an API deprecation communication and telemetry system, we need to ensure that we can accurately track which integrators are using deprecated endpoints, notify them promptly, and confirm that all traffic has ceased before removing the endpoint entirely. This involves a multi-faceted approach combining robust logging, notification systems, and verification mechanisms.

#### Triage

The question is probing how to implement an effective deprecation strategy for APIs while minimizing disruption to integrators and ensuring compliance with business requirements. Key aspects include identifying users, communicating changes effectively, tracking usage over time, and validating the deprecation at sunset.

#### Assumptions
We have a well-defined list of deprecated endpoints.
Integrators are typically developers or organizations that consume our APIs.
Notifications can be sent via email, Slack, or other communication channels.
We use a logging system to capture API request details.
There is a clear definition for "sunset" (e.g., 1 month from deprecation date).

#### First-Principles Solution
Step 1: Logging and Tagging

Implement detailed logging on all API requests, tagging them with metadata such as the consumer’s identifier (if provided by the integrator), endpoint used, timestamp, and any other relevant information. This will allow us to trace back requests during the deprecation period.

API Request
Log & Tag
Record Request Details
Identify Integrator
Send Notification
Wait for Sunset Period
Verify Traffic is Zero
Remove Endpoint
Step 2: Communication Strategy

Notify affected integrators via their preferred communication channels, providing clear instructions on the deprecation timeline and recommended actions. This can be done through automated emails or a dedicated portal.

Step 3: Usage Monitoring

Develop a dashboard to monitor API usage over time. This will allow us to track which endpoints are still being called by which integrators during the deprecation period.

Step 4: Validation at Sunset

At sunset, perform a final verification to ensure no traffic is flowing through the deprecated endpoint. If necessary, conduct additional manual checks or audits to confirm compliance.

#### Closing Summary
and Common Mistakes/Follow-Ups

A robust API deprecation strategy ensures smooth transitions for integrators while minimizing risks associated with removed endpoints. Key challenges include accurate identification of users, timely communication, comprehensive logging, and thorough verification processes. Common pitfalls include insufficient notification timelines, inadequate logging mechanisms, and hasty endpoint removal without proper validation.

For follow-ups, we might discuss integration with monitoring tools, legal considerations, or strategies for handling critical integrators who may need special accommodations during the deprecation period.

**Comparison:**

```
Timestamp	Consumer ID	Endpoint
2023-10-05T14:00:00Z	ABC123	/v1/deprecated-endpoint
2023-10-06T15:00:00Z	XYZ789	/v1/active-endpoint
```

**Code:**

```python
# Example logging function in Python
def log_api_request(consumer_id, endpoint):
    timestamp = datetime.now()
    details = {
        'consumer_id': consumer_id,
        'endpoint': endpoint,
        'timestamp': timestamp
    }
    # Assuming a logger is set up to record this info
    logger.info(details)
```

**Example input / output:**

```text
Input:
- Deprecated endpoint: /v1/deprecated-endpoint
- Sunset date: 2024-01-01

Output:
- Log entry for consumer ABC123 using deprecated endpoint on 2023-10-05T14:00:00Z.
- Notification sent to consumer ABC123 via email informing about deprecation and sunset date.
- Dashboard shows ongoing usage of /v1/deprecated-endpoint by multiple consumers.
- Final verification confirms no traffic through /v1/deprecated-endpoint on 2024-01-01, allowing endpoint removal.
```

**Data:**

```json
{
  "logs": [
    {
      "consumer_id": "ABC123",
      "endpoint": "/v1/deprecated-endpoint",
      "timestamp": "2023-10-05T14:00:00Z"
    },
    {
      "consumer_id": "XYZ789",
      "endpoint": "/v1/active-endpoint",
      "timestamp": "2023-10-06T15:00:00Z"
    }
  ]
}
```

**Flow:**

1. API Request
2. Log & Tag
3. Record Request Details
4. Identify Integrator
5. Send Notification
6. Wait for Sunset Period
7. Verify Traffic is Zero
8. Remove Endpoint

