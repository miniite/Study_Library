---
type: "practice"
difficulty: "Deep"
---

# You're asked to set the retention and tiered-storage policy for the org's telemetry log so incident replay works for weeks but cost stays bounded. What tiering do you standardize, and how do you handle the one compliance team that needs seven years?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Event-time Watermarks](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the retention and tiered-storage policy for the org's telemetry log while ensuring incident replay works for weeks but keeping costs bounded, we need to standardize a multi-tier storage strategy that balances data longevity with financial constraints. We'll start by understanding the core requirements and then delve into the first-principles solution. We're dealing with three main scenarios:

Incident Replay: Requires access to logs from recent weeks to investigate and resolve issues quickly.
Cost Boundedness: Ensures that storage costs remain under control, preventing excessive expenses without compromising incident replay capabilities.
Compliance Requirement: A specific team has a unique need for long-term retention (seven years).

The first-principles approach involves breaking down the problem into manageable components and making informed decisions based on these components.

#### Triage
Core Question: How do we balance short-term accessibility with long-term cost control while meeting specific compliance needs?

#### Assumptions
The telemetry logs are generated in real-time.
Logs need to be accessible for at least two weeks (incident replay requirement).
There's a strict budget limit on storage costs.
Compliance requirements must be met, but with the highest cost efficiency.

#### First-Principles Solution

Define Tiered Storage Levels:

Hot Storage: Logs that need to be immediately accessible for incident replay (e.g., last two weeks). This tier should be kept in a high-performance storage solution.
PLAINTEXT
Hot Storage: Last 2 Weeks, High Performance
Warm Storage: Logs from the previous month to two months. These logs are less frequently accessed but still critical for incident replay.
PLAINTEXT
Warm Storage: Months 1-3, Balanced Performance
Cold Storage: Logs older than three months. These logs are rarely accessed and can be stored in a more cost-effective solution.
PLAINTEXT
Cold Storage: Older Than 3 Months, Cost-Efficient

Implement an Automated Tiering Strategy:

Use a tiering policy that automatically moves logs to the appropriate storage level based on access patterns.
For example, if a log is accessed within the first week after generation, it remains in hot storage. If not, it's moved to warm storage, and eventually cold storage.

Compliance Handling:

The compliance team requires seven years of retention for specific logs. We can address this by creating an exception policy that keeps these logs in a dedicated, high-performance tier.
PLAINTEXT
Compliance Tier: 7 Years, High Performance

Cost Management:

Monitor and adjust the storage tiers based on access patterns to ensure cost efficiency.
Implement budget alerts and automated scaling policies to prevent overspending.

#### Closing Summary
+ Common Mistakes/Follow-Ups
By standardizing a multi-tiered storage strategy, we can meet both short-term accessibility needs for incident replay and long-term cost control. The key is to implement an automated tiering system that dynamically moves logs based on access patterns.

#### Common Mistakes
Failing to recognize the unique compliance requirement and not creating appropriate exceptions.
Over-provisioning storage in high-performance tiers, leading to unnecessary costs.
Not regularly reviewing and adjusting the tiering strategy as log access patterns evolve.

#### Follow-Up
How do you ensure data integrity across different tiers?
What mechanisms will you implement for auditability and reporting on storage usage?

**Comparison:**

| Log ID | Last Access Time     | Storage Tier |
|--------|---------------------|--------------|
| L001   | 2023-07-01T08:00:00Z| Hot          |
| L002   | 2023-06-29T14:00:00Z| Warm         |
| L003   | 2023-05-30T09:30:00Z| Cold         |

**Code:**

```python
# Example Python script to simulate automated tiering logic

def tier_logs(log_id, last_access_time):
    if (datetime.now() - last_access_time).days <= 14:
        return "Hot"
    elif (datetime.now() - last_access_time).days <= 60:
        return "Warm"
    else:
        return "Cold"

log_id = "L12345"
last_access_time = datetime(2023, 7, 1)
storage_tier = tier_logs(log_id, last_access_time)
print(f"Log {log_id} should be stored in: {storage_tier}")
```

**Example input / output:**

```text
Input:
log_id = L001, last_access_time = 2023-07-01T08:00:00Z

Output:
Log L001 should be stored in: Hot

Input:
log_id = L002, last_access_time = 2023-06-29T14:00:00Z

Output:
Log L002 should be stored in: Warm

Input:
log_id = L003, last_access_time = 2023-05-30T09:30:00Z

Output:
Log L003 should be stored in: Cold
```

**Data:**

```json
{
    "logs": [
        {
            "id": "L001",
            "last_access_time": "2023-07-01T08:00:00Z"
        },
        {
            "id": "L002",
            "last_access_time": "2023-06-29T14:00:00Z"
        },
        {
            "id": "L003",
            "last_access_time": "2023-05-30T09:30:00Z"
        }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. ComplianceTier
6. Lifetime >= 7 Years
7. High Performance Storage
8. Log Generated
9. Is Last Access Time <= 14Days?
10. Hot Storage
11. Is Last Access Time > 60 Days?
12. Cold Storage
13. Warm Storage
14. Audit and Report on TieringPolicy

