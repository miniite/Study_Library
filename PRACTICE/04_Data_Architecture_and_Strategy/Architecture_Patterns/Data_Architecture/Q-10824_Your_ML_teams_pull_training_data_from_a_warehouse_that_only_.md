---
type: "practice"
difficulty: "Deep"
---

# Your ML teams pull training data from a warehouse that only retains 90 days, but seasonal models need 3 years. Architect the retention/tiering split between warehouse and lake so long-horizon training is possible without paying warehouse rates for cold data.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Tiered Lake Storage](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a solution where ML teams can train on 3-year-long seasonal models while adhering to the 90-day retention policy of the warehouse, we need to implement a tiered storage architecture that splits data between a hot warehouse and cold lake. This approach ensures efficient use of resources by storing recently accessed data in the warehouse and archiving less frequently used but still critical data in the lake.

#### Triaging the Question

The key aspects being probed are:

Understanding of Data Retention Policies: How to manage data that needs long-term storage while adhering to short-term retention policies.
Cost Optimization: Ensuring minimal cost by not storing cold data redundantly.
Data Accessibility: Ensuring fast access to hot data for immediate use and efficient retrieval from the lake when needed.
Step-by-Step Solution

Define Warehouse (Hot Data)

The warehouse should retain recent, frequently accessed training data for up to 90 days.
This storage is optimized for speed and availability, making it costlier but necessary for real-time or near-real-time operations.

Define Lake (Cold Data)

The lake will store historical data that exceeds the warehouse retention period.
This storage is more cost-effective due to lower bandwidth requirements and less frequent access.

Data Movement Strategy

Automate a process to move data from the warehouse to the lake after 90 days.
Implement efficient data retrieval mechanisms to fetch data from the lake when needed, without incurring high costs.

Archival Tiering (Optional)

#### Cons
ider an archival tier for data older than 3 years if space and cost are a concern.
This tier can store infrequently accessed but still valuable historical data, further optimizing storage costs.

#### Example
Workflow
New training data is loaded into the warehouse as it becomes available.
After 90 days, the data in the warehouse triggers an automated process to move it to the lake.
For long-horizon models, a scheduled job retrieves necessary data from the lake and processes it in the warehouse for training.
Detailed Workflow
Data Ingestion
Data is ingested into the warehouse via a batch or stream processing pipeline.
Retrieval Logic
A background scheduler checks if any data in the warehouse exceeds 90 days of retention.
If so, it triggers an archival process to move the data to the lake.
Lake Access
When training models requiring long-term data (e.g., seasonal models), a specific query or request is made to retrieve required data from the lake.
The system dynamically fetches this data without moving it back to the warehouse, reducing costs.

#### Closing Summary

By archiving cold data to a cost-effective storage tier and using an efficient retrieval mechanism, we can adhere to short-term retention policies while enabling long-horizon training. This approach ensures minimal operational overhead and optimal use of resources.

#### Common Mistakes/Follow-Ups

Incorrect Data Movement: Not setting up automated processes for timely data archiving.
Overhead in Data Retrieval: Failing to implement efficient querying mechanisms that minimize the cost of fetching cold data.
Ignoring Archival Needs: Not considering an archival tier, leading to unnecessary costs or storage limitations.

**Comparison:**

| Storage Tier | Retention Period | Cost Per GB | Access Latency |
|--------------|------------------|-------------|---------------|
| Warehouse    | 90 days          | $0.25       | 1 second      |
| Lake         | 3 years          | $0.04       | 1-3 seconds   |

**Code:**

```python
# Example code for automated data archiving
import schedule
import time

def archive_data():
    # Logic to check and move old data from warehouse to lake
    pass

schedule.every(1).days.do(archive_data)

while True:
    schedule.run_pending()
    time.sleep(60)
```

**Example input / output:**

```text
Input: Training request for a seasonal model requiring data from the past 3 years.
Output: System identifies required data from the lake and retrieves it dynamically, performing necessary transformations in the warehouse.
Request: SELECT * FROM historical_data WHERE timestamp BETWEEN '2021-01-01' AND '2024-12-31';
Response: Data fetched from the lake, transformed in the warehouse for model training.
```

**Data:**

```json
{
  "warehouse": {
    "retentionPeriod": "90 days",
    "costPerGB": "$0.25",
    "accessLatency": "1 second"
  },
  "lake": {
    "retentionPeriod": "3 years",
    "costPerGB": "$0.04",
    "accessLatency": "1-3 seconds"
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Data Ingestion
5. Is Data Old?
6. Lake Archival
7. Warehouse Retention Check
8. Warehouse RetentionExceeded?
9. Archive to Lake
10. Traffic Light for TrainingModels
11. Retrieve Data from Warehouse
12. Dynamically Retrieve fromLake

