---
type: "practice"
difficulty: "Expert"
---

# Walk me through how you build a multi-year database capacity model for a platform growing 40% YoY across compute, storage, IOPS, and connections — what leading indicators do you anchor it to?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Capacity Planning and Throughput](<../../../../01_Databases_and_SQL/Database_Operations/Capacity_Planning/Capacity_Planning_and_Throughput.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a multi-year database capacity model for a growing platform, we need to systematically assess and project various resource requirements such as compute, storage, IOPS, and connections over time. The key is to use leading indicators that directly correlate with the growth of the platform and ensure the model is robust enough to handle uncertainties.

#### Triage

The core of this question revolves around understanding how to forecast database capacity needs for a growing system using reliable metrics as anchor points. We need to:

Identify the relevant performance metrics.
Establish baseline data from current operations.
Apply growth projections based on historical trends and future forecasts.
Validate assumptions with real-world data.
First Principles
Understand the Current State: Gather detailed information about the current database environment, including current resource utilization, throughput, and efficiency metrics.
Growth Projections: Use historical growth rates (40% YoY) as a starting point for future projections.
Leading Indicators: Identify key performance indicators (KPIs) such as transactions per second, query complexity, and data volume that can serve as proxies for resource requirements.
Scenario Analysis: Develop multiple scenarios to account for different growth rates, operational changes, or technological advancements.
Detailed Steps

Gather Current Metrics

Compute: Measure CPU usage, query execution time, and transaction throughput.
Storage: Track data volume, backup frequency, and retention policies.
IOPS: Monitor disk I/O operations per second for read and write operations.
Connections: Count concurrent connections and session duration.

Establish Baseline Data

Collect historical performance metrics over the past 6-12 months to identify trends and variability.
Use this data to establish baseline utilization rates and efficiency metrics.

Apply Growth Projections

Project future growth in compute, storage, IOPS, and connections using a compound annual growth rate (CAGR) of 40%.
Adjust for seasonal variations if applicable.

Identify Leading Indicators

Use KPIs like transactions per second to predict the number of database servers needed.
Track data volume growth to forecast storage needs and backup frequency.
Monitor query complexity metrics to anticipate IOPS requirements.
Keep an eye on concurrent connections to ensure sufficient connection pool capacity.

Scenario Analysis

Develop best-case, worst-case, and most-likely scenarios for future growth rates.

#### Cons
ider changes in technology, such as the adoption of new database technologies or optimizations that could impact resource utilization.
Validate these scenarios with expert input and historical data trends.

Validation and Refinement

Regularly validate the model against actual performance metrics to ensure accuracy.
Refine projections based on real-world observations and feedback from operations teams.

#### Closing Summary

By leveraging leading indicators such as transactions per second, query complexity, and data volume growth, we can build a robust multi-year database capacity model that accounts for future growth rates. This approach ensures that the model remains flexible enough to adapt to changing conditions while providing clear guidance on resource needs.

#### Common Mistakes/Follow-Ups
Overreliance on Historical Data: Assuming past trends will continue without considering new technologies or operational changes.
Lack of Scenario Analysis: Failing to consider worst-case and best-case scenarios, which can lead to under-provisioning or over-provisioning resources.
Ignoring User Behavior Changes: Not accounting for potential shifts in user behavior that could impact resource requirements.

**Comparison:**

```
Year	Volume [GB]
1	700
2	1,050
3	1,580
```

**Code:**

```python
def calculate_future_storage_size(current_volume_gb, growth_rate, years):
    return current_volume_gb * (1 + growth_rate) ** years

future_storage = calculate_future_storage_size(500, 0.4, 3)
print(f"Future Storage Size: {future_storage} GB")
```

**Example input / output:**

```text
Input:
- Current Volume: 500 GB
- Growth Rate: 40% (CAGR)
- Years: 3

Output:
- Future Storage Size: 796.264 GB
```

**Data:**

```json
{
  "currentVolumeGB": 500,
  "growthRate": 0.4,
  "years": 3
}
```

**Flow:**

1. Current Metrics Collection
2. Gather Historical Data
3. Identify Leading Indicators
4. Project Future Growth
5. Scenario Analysis
6. Validate Model withOperations

