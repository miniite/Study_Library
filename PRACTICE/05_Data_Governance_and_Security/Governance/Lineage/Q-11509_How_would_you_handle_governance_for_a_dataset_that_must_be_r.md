---
type: "practice"
difficulty: "Deep"
---

# How would you handle governance for a dataset that must be retained for 7 years for compliance but only 90 days is hot? Design the tiering.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Ownership](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this data governance and lineage challenge, we need to design a tiered storage solution that retains data for 7 years while ensuring that frequently accessed data (hot data) is stored in fast, cost-effective tiers with shorter retention periods. Here’s how we can approach this:

#### Overview
We will implement an automated tiering strategy that categorizes data based on access frequency and moves it between different storage tiers to optimize costs while maintaining compliance.
Restate the Question & Probing Points: The core question is about designing a tiered storage architecture for data retention policies. The interviewer is probing our understanding of data classification, automation, cost optimization, and compliance adherence.

#### First-Principles Solution
Data Classification: Identify "hot" vs. "cold" data based on access patterns.
Tiering Strategy: Implement a tiered storage solution with different retention periods for hot and cold data.
Automation & Monitoring: Use automated tools to monitor and manage the movement of data between tiers.

Let's break down each step in detail:

1. Data Classification
Hot Data (90 Days): This includes recent transactions, active queries, or any data that needs quick access within a short period.
Cold Data (7 Years): This includes historical data that does not require frequent access but must be retained for regulatory compliance.
2. Tiering Strategy

We will use three storage tiers:

Tier 1 - Hot Storage: Store the most frequently accessed data with a 90-day retention period.
Tier 2 - Warm Storage: Store data that is less frequently accessed but needs to be retrieved within days or weeks, with a longer retention period (e.g., 30-60 days).
Tier 3 - Cold Storage: Archive older data with the required 7-year retention period.
3. Automation & Monitoring

Implement an automated tiering system using tools like AWS Glue, Apache Airflow, or custom scripts:

Access Pattern Analysis: Use logs to track access patterns and automatically reclassify data based on recency.
Movement Rules: Define rules for moving data between tiers (e.g., move from Tier 2 to Tier 3 after a certain period).
Monitoring & Alerts: Set up monitoring and alerts to ensure compliance and optimize cost by identifying underutilized or idle storage.

#### Example
Workflow:
Initial Classification:
Identify hot data based on recent access times.
Move this data to Tier 1 with a 90-day retention period.
Monitoring & Re-Classification:
Periodically (daily/weekly) analyze logs to re-evaluate the access patterns of data in Tier 2.
If a piece of data is accessed, move it back to Tier 1 for another 90 days.
Data Archival:
Once hot data moves out of Tier 1, automatically archive it to Tier 2 with a longer retention period.
Long-Term Storage:
Periodically (weekly/monthly) reclassify data in Tier 2 and move older data to Tier 3 for long-term storage.

#### Closing Summary
& Common Mistakes/Follow-Ups
Ensure compliance by adhering strictly to the required retention periods.
Optimize costs through automated tiering, ensuring that hot data remains accessible while cold data is stored efficiently.
Regularly review access patterns and adjust classification rules as necessary.
Automate monitoring and alerts to quickly identify any deviations from expected behavior.

#### Common mistakes
include overcomplicating the solution or failing to automate processes, leading to increased manual effort and potential errors. Always balance compliance with cost efficiency through thoughtful tiering strategies.

**Comparison:**

```
Data Type	Retention Period	Tier
Hot	90 days	Tier 1 (Hot Storage)
Warm	30-60 days	Tier 2 (Warm Storage)
Cold	7 years	Tier 3 (Cold Storage)
```

**Code:**

```python
# Example Python script for data classification and movement
def classify_data(access_logs):
    hot_data = []
    warm_data = []
    cold_data = []

    for log in access_logs:
        if log.recent_access:
            hot_data.append(log.data)
        elif log.frequency >= 10:  # Example threshold
            warm_data.append(log.data)
        else:
            cold_data.append(log.data)

    return hot_data, warm_data, cold_data

def move_to_tier(data, tier):
    # Simulate moving data to different tiers based on retention periods
    for item in data:
        if '90_days' in tier:
            # Store in Tier 1 with 90-day retention
            store_hot(item)
        elif '30-60_days' in tier:
            # Store in Tier 2 with 30-60 day retention
            store_warm(item)
        else:
            # Archive to Tier 3 for long-term storage
            archive(item)

# Example usage
access_logs = [
    {'data': 'user1_transactions', 'recent_access': True, 'frequency': 5},
    {'data': 'user2_reports', 'recent_access': False, 'frequency': 0}
]
hot_data, warm_data, cold_data = classify_data(access_logs)
move_to_tier(hot_data, ['90_days'])
move_to_tier(warm_data, ['30-60_days'])
move_to_tier(cold_data, ['7_years'])
```

**Example input / output:**

```text
Input:
- Access logs for user1_transactions and user2_reports

Output:
- user1_transactions moved to Tier 1 with 90-day retention
- user2_reports archived to Tier 3 for long-term storage
```

**Data:**

```json
{
  "access_logs": [
    {
      "data": "user1_transactions",
      "recent_access": true,
      "frequency": 5
    },
    {
      "data": "user2_reports",
      "recent_access": false,
      "frequency": 0
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Frequency < 10
4. Frequency >= 10
5. Access Logs
6. Is Data Hot?
7. Store in Tier 1
8. Check Frequency
9. Archive to Tier 3
10. Store in Tier 2

