---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd decide the cost-optimal storage class strategy for a data lake — hot for recent partitions, infrequent-access for older, archive for cold — given that a bad lifecycle rule either bloats cost or makes a partition un-queryable when an analyst needs history. Design the automatic-yet-safe transition policy.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Retention Costs and Compliance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Retention_Costs_and_Compliance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide on the cost-optimal storage class strategy for a data lake, we need to balance the need for queryability with storage costs. The goal is to automatically and safely transition partitions based on their recency, ensuring that recent data remains accessible while older data can be archived at lower costs without disrupting historical analysis. This involves understanding the lifecycle of data within the data lake and setting up a tiered storage strategy.

#### Triaging the Question

The interviewer is probing how you would implement a comprehensive storage lifecycle management (SLM) policy for a data lake, focusing on minimizing cost while maintaining queryability. Key considerations include:

Data Recency: Determining when partitions are recent enough to require hot storage.
Historical Data: Ensuring that older partitions can be moved to cheaper storage tiers without affecting querying capabilities.
Cost Bloat: Avoiding unnecessary costs by not over-retaining data in expensive storage classes.
Queryability: Maintaining the ability to query any partition as needed, even if it’s been transitioned.

#### First Principles Solution

Data Partitioning and Metadata Management:

Data Lakes typically store large volumes of structured or semi-structured data, often organized into partitions based on timestamps (e.g., daily or monthly).
Metadata Storage: Maintain a metadata layer to track partition identifiers, their lifecycle status, and any associated query history.

Tiered Storage Strategy:

Hot Storage Class: For recent partitions where high query frequency is expected.
Infrequent Access (IA) Storage Class: For older partitions that are not queried as frequently but still need to be accessible for historical analysis.
Archive Storage Class: For very old data that can be stored at the lowest cost and least frequent access.

Automated Transition Policy:

Define a policy that automatically transitions partitions based on their last query date or age.
Ensure that queries are not blocked by transitional state changes.

#### Implementation
Steps

Metadata Tracking:

SQL
CREATE TABLE data_partitions (
    partition_id INT PRIMARY KEY,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    lifecycle_class VARCHAR(50),
    last_query_ts TIMESTAMP DEFAULT NULL
);

This schema tracks each partition's details and its current lifecycle class.

Lifecycle Rules:

Define rules to transition partitions from one storage tier to another based on the last_query_ts.

#### Example
rule:
SQL
UPDATE data_partitions
SET lifecycle_class = 'IA'
WHERE last_query_ts < (CURRENT_DATE - INTERVAL 90 DAYS) AND lifecycle_class = 'Hot';
Schedule these updates periodically using a cron job or AWS CloudWatch Events.

Querying with Safeguards:

Ensure that queries can still access data in the transitional state by temporarily moving it back to Hot storage.
SQL
CREATE OR REPLACE VIEW queryable_data AS
SELECT * FROM (
    SELECT *, CASE
        WHEN lifecycle_class = 'IA' AND last_query_ts >= (CURRENT_DATE - INTERVAL 90 DAYS) THEN 'Hot'
        ELSE lifecycle_class END as effective_lifecycle_class
    FROM data_partitions
) t;

This view ensures that queries always see the latest lifecycle status, temporarily restoring Hot storage for recently queried partitions.

Monitoring and Alerting:

Set up monitoring to alert on any issues with partition queryability or cost.
BASH
aws cloudwatch put-metric-alarm --alarm-name HotStorageIssues \
    --metric-name PartitionQueryErrors \
    --namespace AWS/Logs \
    --threshold 10 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 1 \
    --statistic Sum \
    --period 3600 \
    --dimensions "Name=PartitionId,Value=<partition_id>"

#### Closing Summary
and Common Mistakes

In summary, the key is to balance queryability with cost by implementing a tiered storage strategy with automated lifecycle rules. The solution involves metadata management, dynamic transition policies, and robust querying safeguards.

#### Common Mistakes

Overlooking Query Performance: Ensuring that transitioning partitions does not impact query performance.
Ignoring Cost Bloat: Failing to define clear thresholds for moving data between tiers.
Lack of Monitoring: Not setting up alerts to monitor storage classes and query errors.

**Comparison:**

```
FLOWCHART · MERMAID
pie title Lifecycle Status
    "Hot" : 50%
    "IA" : 40%
    "Archive" : 10%
```

**Code:**

```sql
CREATE TABLE data_partitions (
    partition_id INT PRIMARY KEY,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    lifecycle_class VARCHAR(50),
    last_query_ts TIMESTAMP DEFAULT NULL
);

UPDATE data_partitions
SET lifecycle_class = 'IA'
WHERE last_query_ts < (CURRENT_DATE - INTERVAL 90 DAYS) AND lifecycle_class = 'Hot';

CREATE OR REPLACE VIEW queryable_data AS
SELECT * FROM (
    SELECT *, CASE
        WHEN lifecycle_class = 'IA' AND last_query_ts >= (CURRENT_DATE - INTERVAL 90 DAYS) THEN 'Hot'
        ELSE lifecycle_class END as effective_lifecycle_class
    FROM data_partitions
) t;
```

**Example input / output:**

```
Partition ID	Start Date	End Date	Lifecycle Class	Last Query TS
1	2023-01-01	2023-01-31	Hot	2023-06-01
2	2023-02-01	2023-02-28	IA	2023-05-01
3	2023-03-01	2023-03-31	IA	null

After lifecycle update and view creation:

SQL
Copy
SELECT * FROM queryable_data;

Output (assuming the partitions are updated as per rules):

Partition ID	Start Date	End Date	Lifecycle Class	Last Query TS
1	2023-01-01	2023-01-31	Hot	2023-06-01
2	2023-02-01	2023-02-28	IA	2023-05-01
3	2023-03-01	2023-03-31	IA (temp Hot)	null
```

**Data:**

```json
{
  "data_partitions": [
    {"partition_id": 1, "start_date": "2023-01-01", "end_date": "2023-01-31", "lifecycle_class": "Hot", "last_query_ts": "2023-06-01"},
    {"partition_id": 2, "start_date": "2023-02-01", "end_date": "2023-02-28", "lifecycle_class": "IA", "last_query_ts": "2023-05-01"},
    {"partition_id": 3, "start_date": "2023-03-01", "end_date": "2023-03-31", "lifecycle_class": "IA", "last_query_ts": null}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Partition
6. Metadata Update
7. Lifecycle Transition
8. Check If Last Query TS is Null?
9. Set Lifecycle Class to IA(temporarily)
10. Set Lifecycle Class to Archive

