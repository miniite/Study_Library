---
type: "practice"
difficulty: "Deep"
---

# Your team wants hourly partitions for freshness but finance wants monthly for cost — how do you design a tiered partition layout that satisfies both without duplicating data?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partitioning and Bucketing Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we need to design a storage layout that supports both hourly partitions for data freshness and monthly partitions for cost efficiency. The solution involves using a hybrid approach with partitioning on both time dimensions—hourly and monthly—to ensure the data is accessible in the desired granularity without duplicating it.

#### Triage
Understanding the Problem

The team needs hourly partitions to maintain data freshness, but finance requires monthly partitions due to storage costs. Our goal is to design a system that can meet these requirements efficiently without redundant storage.

#### First-Principles Solution
Understanding Partitioning: Partitions are used to split large datasets into smaller, more manageable chunks based on specific criteria.
Hybrid Partitioning Approach: We will use a combination of hourly and monthly partitions. This approach allows us to leverage the benefits of both partition types while minimizing redundancy.
Design Details

Hourly Partitions for Freshness:

Use hourly partitions to ensure data is immediately available at a high level of granularity.

#### Example
year=2023/month=05/day=01/hour=h00-11.

Monthly Buckets for Cost Efficiency:

Use monthly buckets to group the hourly data into manageable chunks for cost analysis.

#### Example
year=2023/month=05 contains all hours from May 2023.

#### Implementation

To implement this, we can use a combination of directory structures and storage systems like Amazon S3 or Google Cloud Storage. Here’s an example structure:

CODE
/year={year}/month={month}/day={day}/hour={hour}

For example:

2023/05/01/h00 for January 1, 2023, at 00:00.
2023/05/monthly_summary for aggregated monthly data.
Sample Schema
JSON
{
  "directoryStructure": {
    "{year}": [
      "{month}",
      "{day}/hour={hour}"
    ],
    "{month}": [
      "monthly_summary"
    ]
  }
}
Dry-Run Example

Assume we have the following input data:

2023/05/01/h00 (January 1, 2023, at 00:00).
2023/05/monthly_summary.

For this data, both hourly and monthly partitions are created. The hourly partition is stored directly under the date-time directory, while the monthly summary is aggregated and stored in a separate folder.

#### Closing Summary

By using a hybrid approach with both hourly and monthly partitions, we can meet the needs of both the team for freshness and finance for cost efficiency without duplicating data. This design ensures that the system remains scalable and efficient.

#### Common Mistakes/Follow-Ups
Duplicate Storage: Ensuring that no duplicate storage is created between hourly and monthly partitions.
Aggregation Logic: Properly implementing aggregation logic to avoid redundancy and ensure accuracy.
Query Performance: Optimizing queries to efficiently retrieve data from the appropriate partition levels.

**Comparison:**

| Year | Month | Day   | Hour     | Partition Path                          |
|------|-------|-------|----------|-----------------------------------------|
| 2023 | 05    | 01    | h00      | 2023/05/01/hour=h00                     |
| 2023 | 05    | monthly_summary |       | 2023/05/monthly_summary                 |

**Code:**

```python
# Example Python code for generating directory paths

def generate_partition_path(year, month, day, hour):
    return f"{year}/{month}/{day}/hour={hour}"

partition_path = generate_partition_path("2023", "05", "01", "h00")
print(partition_path)
```

**Example input / output:**

```text
Input:
- Data for January 1, 2023, at 00:00.

Output:
- Hourly partition path: `2023/05/01/hour=h00`.
- Monthly summary path: `2023/05/monthly_summary`.

Aggregated monthly data is stored in the `monthly_summary` folder.
```

**Data:**

```json
{
  "directoryStructure": {
    "2023": [
      "05",
      "06"
    ],
    "05": [
      "01/hour=h00",
      "01/hour=h01",
      "monthly_summary"
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Input Data
6. Is Hourly Partition?
7. Store in Hourly Path
8. Is Monthly Summary Needed?
9. Store in Monthly SummaryFolder

