---
type: "practice"
difficulty: "Expert"
---

# How do you architect a 'restore SLA' contract with downstream teams — what latency and cost do you commit to for pulling archived data back, and how do you enforce it?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Storage Tiering Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Storage_Tiering_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an effective "restore SLA" contract for pulling archived data back from storage tiers like Hot/Cold/Glacier, we need to balance latency requirements with cost optimization. This involves defining clear Service Level Agreements (SLAs) with downstream teams and implementing robust enforcement mechanisms. The key steps are: 1) identifying the appropriate tiering policies based on access patterns, 2) setting realistic SLA commitments for data restore times, 3) leveraging automation to enforce these agreements, and 4) continuously monitoring performance and adjusting as needed.

TL;DR: Architect a storage tiering strategy by defining clear SLAs with downstream teams, using automated enforcement mechanisms, and continuously optimizing based on usage patterns.

First, let's break down the question. The interviewer is probing how you would define and enforce restore SLAs that ensure data can be recovered within acceptable latencies while managing costs effectively. This involves understanding the different storage tiers (Hot, Cold, Glacier) and their respective characteristics, as well as considering the trade-offs between access speed and cost.

We start by categorizing our data into tiers based on usage patterns:

Hot Tier: Frequent access to recently modified or critical data.
Cold Tier: Less frequent access to older but still relevant data.
Glacier Tier: Infrequent access to very old data, which we need to keep for compliance reasons.

Next, we define SLAs for each tier. For example:

Hot Tier: Ensure 99.9% of data can be restored within 5 minutes with a maximum penalty in the event of latency breaches.
Cold Tier: Aim for 80-90% availability within an hour, with penalties for higher latencies.
Glacier Tier: Achieve 100% restore within 4 hours, but with no strict financial penalties.

To enforce these SLAs, we use a combination of automated monitoring and alerts:

Yes
No
Within SLA
Exceeded SLA
Yes
No
Data Request
Is the data in Hot Tier?
Immediate S3 GET
Check Cold Tier Latency
Return data from Cold Tier
Trigger Alert, Re-index to ColdTier
Is the data in Glacier Tier?
Start S3 Glacier Restore Job(up to 48-72 hours)
Return data from Hot/ColdTier

This workflow ensures that data is moved up the tier stack only when necessary, minimizing costs while still meeting access needs.

#### Common mistakes
include:

Overestimating cold and glacier access rates, leading to unnecessary data movement.
Failing to automate alerting and remediation processes, causing manual latencies.
Not regularly reviewing and updating SLAs based on actual usage patterns.

In summary, a well-architected restore SLA involves clear tiering policies, realistic latency commitments, automated enforcement mechanisms, and continuous optimization. By doing so, we can ensure that data is accessible when needed without incurring excessive costs.

**Comparison:**

```
Tier	Latency SLA (%)	Restore Cost ($/GB)	Max Retries
Hot	[95, 100]	0.02	3
Cold	[80, 90]	0.01	5
Glacier	[75, 100]	0.001	10
Tier	Action
Hot	Immediate S3 GET
Cold	Check Latency, Return Data if Available, else Trigger Restore Job
Glacier	Start S3 Glacier Restore Job
```

**Code:**

```python
def check_restore_latency(tier, request_time):
    if tier == 'Hot':
        max_latency = 5 * 60  # 5 minutes
    elif tier == 'Cold':
        max_latency = 3600  # 1 hour
    else:  # Glacier
        max_latency = 48 * 3600 + 72 * 3600  # 4 hours (S3 Glacier restore time)

    current_time = datetime.now()
    elapsed_time = (current_time - request_time).total_seconds()

    if elapsed_time > max_latency:
        return False
    else:
        return True
```

**Example input / output:**

```text
Input:
- Data request for archived data in Cold Tier

Output:
- If the data is available in Hot or Cold Tier, return it immediately.
- If not, trigger a S3 Glacier restore job and monitor its progress.
```

**Data:**

```json
{
  "tier_policies": {
    "Hot": {
      "latency_slas": [95, 100],
      "restore_cost": 0.02,
      "max_retries": 3
    },
    "Cold": {
      "latency_slas": [80, 90],
      "restore_cost": 0.01,
      "max_retries": 5
    },
    "Glacier": {
      "latency_slas": [75, 100],
      "restore_cost": 0.001,
      "max_retries": 10
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Within SLA
4. Exceeded SLA
5. Yes
6. No
7. Data Request
8. Is the data in Hot Tier?
9. Immediate S3 GET
10. Check Cold Tier Latency
11. Return data from Cold Tier
12. Trigger Alert, Re-index to ColdTier
13. Is the data in Glacier Tier?
14. Start S3 Glacier Restore Job(up to 48-72 hours)
15. Return data from Hot/ColdTier

