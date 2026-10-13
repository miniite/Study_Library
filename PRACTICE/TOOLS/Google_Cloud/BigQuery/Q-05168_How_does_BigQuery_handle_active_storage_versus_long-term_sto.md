---
type: "practice"
difficulty: "Intermediate"
---

# How does BigQuery handle active storage versus long-term storage pricing?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ BigQuery Cost Control](<../../../../TOOLS/Google_Cloud/BigQuery/BigQuery_Cost_Control.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

BigQuery is a fully-managed, highly scalable data warehouse that handles different types of storage based on the nature and lifecycle of your data. The key distinction lies between "active storage" and "long-term storage," which are priced differently to reflect their usage patterns.

In essence, BigQuery charges for active storage when you query or analyze recent data frequently, while long-term storage is used for older data that isn’t accessed as often, keeping it cheaper. This pricing model ensures cost efficiency by encouraging the separation of hot and cold data.

#### Triage

The question asks about how BigQuery manages different types of storage based on usage.
The interviewer likely wants to understand your grasp of BigQuery’s pricing structure and its underlying logic.

#### Assumptions
include understanding that BigQuery has dynamic pricing based on query frequency and storage access patterns.

#### First-principles Solution

Understanding Active Storage:
Active storage refers to data that is accessed frequently for real-time analysis, reporting, or querying. It's stored in a more expensive tier of storage because it needs to be readily available.
Long-Term Storage:
Long-term storage is designed for archived data that is infrequently accessed but still needs to be kept for regulatory compliance or historical purposes. This tier is cheaper than active storage, making it suitable for data that doesn't require immediate access.

To handle these two types of storage effectively in BigQuery:

Partitioning and Clustering: Use partitioning to store recent data in more expensive tiers (active storage) and older data in less expensive tiers (long-term storage). Clustering helps optimize queries by organizing the data based on frequently accessed columns.
Time-Based Partitioning: Implement time-based partitioning strategies, such as monthly or yearly partitions, to automatically move old data into cheaper long-term storage.
Storage Classes:
Standard Storage: Default tier for most use cases. Optimized for low-latency queries and frequent access.
Coldline Storage: For data that is accessed less than once a month but needs quick access when needed.
Linetier Storage: Designed for highly accessible archival data where the cost of moving data back to Standard or Coldline tiers is important.

#### Closing Summary
BigQuery efficiently manages storage costs by separating active and long-term storage. This separation allows users to optimize their queries while keeping older data cheaper, ensuring that they can handle varying access patterns without incurring unnecessary expenses. Understanding this model helps in designing cost-effective BigQuery environments.

#### Common Mistakes/Follow-ups

A common mistake is not recognizing the difference between Standard Storage and Coldline/Linetier storage.
Interviewers might ask about specific use cases where these different storage tiers are most beneficial, or how to migrate data between tiers to optimize costs further.

**Comparison:**

```
Storage Class	Description
Standard	Optimized for low-latency queries and frequent access.
Coldline	For data that is accessed less than once a month but needs quick access.
Linetier	Designed for highly accessible archival data with cost optimization.
```

**Code:**

```sql
-- Example of partitioning and clustering in BigQuery
CREATE TABLE my_dataset.my_table (
  id INT64,
  event_time TIMESTAMP
)
PARTITION BY DATE(event_time)
CLUSTER BY id;

-- Migrating old data to Coldline storage
ALTER TABLE my_dataset.my_table SET OPTIONS(
  expiration_timestamp = TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 1 YEAR)
);
```

**Example input / output:**

```text
Input:
- Recent data needs frequent access and low-latency queries.
- Older data should be archived but accessed occasionally.

Output:
1. Partition recent data into daily or weekly partitions for frequent access.
2. Cluster by a primary key to optimize query performance on active data.
3. Set up Coldline storage for data older than 6 months with infrequent access.
4. Use Linetier storage for highly accessible archival data that needs cost optimization.
```

**Data:**

```json
{
  "storage_classes": [
    {
      "name": "Standard",
      "description": "Optimized for low-latency queries and frequent access."
    },
    {
      "name": "Coldline",
      "description": "For data that is accessed less than once a month but needs quick access when needed."
    },
    {
      "name": "Linetier",
      "description": "Designed for highly accessible archival data where the cost of moving data back to Standard or Coldline tiers is important."
    }
  ]
}
```

**Flow:**

1. Data Management
2. Yes
3. No
4. Yes
5. No
6. Data Freshness
7. Is the data frequentlyaccessed?
8. Use Standard Storage andClustering
9. Is the data older than 6months?
10. Coldline Storage
11. Linetier Storage
12. Migrate to Coldline or Linetier

