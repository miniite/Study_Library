---
type: "practice"
difficulty: "Deep"
---

# Your org's DynamoDB tables have accumulated 200+ GSIs across services with nobody tracking which are used. Design a program to audit, deprecate, and consolidate them safely, and quantify the write-amplification savings to justify the effort to leadership.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ DynamoDB Modeling and Capacity](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To address the issue of unmanaged Global Secondary Indexes (GSIs) in DynamoDB, we will design a program that audits all existing GSIs across services, identifies unused or redundant ones, deprecates them safely, and consolidates the remaining indexes. This process ensures minimal disruption to applications while quantifying potential write-amplification savings to justify the effort to leadership.

#### Triaging the Question

The core of this question revolves around identifying and managing unnecessary GSIs in DynamoDB, which can help reduce write amplification costs. The interviewer is probing your ability to design a comprehensive solution that covers:

Audit: Identify unused or redundant GSIs.
Deprecation: Safely deprecate these indexes without disrupting existing services.

#### Cons
olidation: Consolidate remaining indexes for better performance and cost efficiency.
Cost Savings Calculation: Quantify the potential savings to justify the effort.

#### Assumptions
DynamoDB tables are being used in a multi-service environment, with different teams managing their own tables.
There is no centralized tracking of GSIs across services.
The program should be designed for minimal disruption and safe operation.
Cost savings can be quantified based on current write operations and potential reduction.
Step-by-Step Solution
1. Data Collection

First, collect detailed metadata about all DynamoDB tables in use across the organization. This includes:

JSON
{
    "TableName": "ServiceA",
    "GlobalSecondaryIndexes": [
        {
            "IndexName": "GSI_0",
            "KeySchema": [...],
            "ProvisionedThroughput": {...},
            "IndexStatus": "ACTIVE"
        },
        // ... other GSIs
    ],
    "ItemCount": 12345,
    "ReadCapacityUnits": 100,
    "WriteCapacityUnits": 50
}
2. Audit Logic

Develop an audit algorithm that:

Tracks Usage: Monitor write operations to each GSI over a predefined period (e.g., last month).
Identify Redundancy: Compare GSIs with similar key schemas and index names across different services.
Deprecation Criteria: Define criteria for deprecation, such as low usage or redundant indexes.
3. Safe Deprecation

When a GSI is identified for deprecation:

Pre-Migration: Temporarily redirect all write traffic to another active GSI if possible.
Update Application Code: Gradually update application code to use alternative GSIs.
Monitor Post-Deprecation: Ensure no immediate issues arise after deprecation.
4. Consolidation

After deprecating redundant indexes, consolidate the remaining GSIs:

Optimize Key Schemas: Simplify and optimize key schemas for better performance.
Provision Throughput Adjustments: Dynamically adjust read/write capacity based on actual usage patterns.
5. Cost Savings Calculation

Estimate write amplification savings by comparing current write costs with projected costs after deprecation and consolidation:

PYTHON
def calculate_savings(current_cost, projected_cost):
    return (current_cost - projected_cost) / current_cost * 100

# Example usage:
current_cost = 5000  # $ in monthly charges
projected_cost = 2000  # $ after optimizations
savings_percentage = calculate_savings(current_cost, projected_cost)

#### Closing Summary
and Common Mistakes/Follow-Ups

By designing a comprehensive audit and management program for GSIs, we ensure that write operations are optimized, reducing costs while maintaining application stability. The key is to balance thoroughness with minimal disruption.

#### Common Mistakes

Ignoring the actual usage patterns during auditing.
Not properly managing dependencies between services when deprecating or consolidating indexes.
Overlooking long-term cost savings versus short-term effort.

#### Follow-Ups
What tools would you use for auditing?
How would you handle multi-region DynamoDB tables in this process?
Can you provide an example of how you would implement the safe deprecation step?

**Comparison:**

| TableName | IndexName   | ItemCount  | ReadCapacityUnits | WriteCapacityUnits |
|-----------|-------------|------------|-------------------|--------------------|
| ServiceA  | GSI_0       | 12345      | 100               | 50                 |

**Code:**

```python
def monitor_write_operations(table_name):
    # Code to monitor write operations for a given table name
    pass

def calculate_usage(index_name, start_date, end_date):
    # Code to calculate usage based on DynamoDB metrics
    return 0.5  # Placeholder value
```

**Example input / output:**

```text
# Example input for monitoring write operations
table_name = "ServiceA"

# Example output of usage calculation
usage_percentage = calculate_usage("GSI_0", "2023-10-01", "2023-11-01")
print(f"Usage percentage: {usage_percentage}")
```

**Data:**

```json
{
    "TableName": "ServiceA",
    "GlobalSecondaryIndexes": [
        {
            "IndexName": "GSI_0",
            "KeySchema": [...],
            "ProvisionedThroughput": {...},
            "IndexStatus": "ACTIVE"
        }
    ]
}
```

**Flow:**

1. Start
2. Monitor Write Operations
3. Identify Unused/RedundantGSIs
4. Deprecate or Consolidate GSIs
5. Migrate Traffic & Update Code
6. Post-Migration Monitoring
7. Calculate Cost Savings
8. End

