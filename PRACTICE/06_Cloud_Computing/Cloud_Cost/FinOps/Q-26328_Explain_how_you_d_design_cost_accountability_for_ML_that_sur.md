---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design cost accountability for ML that survives constant reorgs, where tags keyed to model-owning teams break every quarter. Architect an attribution scheme anchored to durable model/product identity rather than mutable org charts, and how you'd backfill history when a team splits.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Funding Models and Incentives](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Funding_Models_and_Incentives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design cost accountability for ML that survives frequent organizational reorganizations, we need to create an attribution scheme based on durable model/product identity rather than mutable org charts. This involves designing a system that can handle dynamic changes in team ownership and backfilling historical costs when a team splits or merges. Here’s how you’d approach this:

#### Overview
The core idea is to decouple cost allocation from transient organizational structures, ensuring robust and accurate cost tracking even as teams change. We’ll use unique identifiers for models/products and develop a method to retroactively update cost records based on these IDs.

#### Triaging the Question

What’s Being Probed: The interviewer is looking for an understanding of cloud cost governance principles, dynamic systems design, and the ability to handle complex business changes.

#### Assumptions
We assume the organization uses a modern cloud platform that supports tagging and budgeting. We also assume there are mechanisms in place for financial tracking and reporting.

#### First Principles Solution

Step 1: Unique Model/Product Identifiers: Create a unique, unchanging identifier (UUID or product code) for each ML model/product. These identifiers should be embedded within the cloud resources associated with the models.

Step 2: Tagging and Metadata Storage: Use cloud tags to store information about the owning team and other relevant metadata, but ensure that these tags are non-mutating over time. For example, you could use a combination of tags and metadata stored in an external database or a managed service like AWS CloudTrail.

Step 3: Cost Attribution Rules: Define rules for attributing costs to specific models/products based on the unique identifiers and metadata. These rules should be flexible enough to adapt to organizational changes but robust enough to remain consistent over time.

Step 4: Historical Data Backfilling: Develop a process to backfill historical cost data when teams split or merge. This involves identifying the original ownership of resources and reassigning costs based on the new identifier scheme.

Step 5: Continuous Monitoring and Reporting: Implement real-time monitoring and reporting to ensure that costs are accurately tracked and updated as organizational changes occur.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The solution involves creating a durable model/product identity system that can handle dynamic organizational changes without breaking cost tracking. This requires careful design of tagging strategies, attribution rules, and historical data backfilling processes.

#### Common mistakes
include relying too heavily on mutable org charts for cost allocation or failing to implement robust backfill mechanisms.

#### Example
Implementation:

Model/Product Creation
Tagging with Unique ID
Storing Metadata in ExternalDatabase
Defining Cost Attribution Rules
Real-Time Monitoring &Reporting
Team Split/Merge
Backfill Historical Costs

**Comparison:**

| Step | Action | Description |
|------|--------|-------------|
| 1    | Unique Model/Product Identifiers | Generate and embed unique IDs in cloud resources. |
| 2    | Tagging and Metadata Storage | Use tags to store metadata about the owning team. |
| 3    | Cost Attribution Rules | Define rules for cost attribution based on unique IDs. |
| 4    | Historical Data Backfilling | Develop a process to reassign costs after organizational changes. |
| 5    | Continuous Monitoring & Reporting | Ensure real-time tracking and reporting of costs. |

**Code:**

```python
# Example code snippet for tagging and metadata storage in AWS

import boto3

def tag_resource(resource_id, unique_id):
    client = boto3.client('resource')
    response = client.tag_resource(
        Resource=resource_id,
        Tags=[
            {'Key': 'UniqueID', 'Value': unique_id},
        ]
    )
    return response
```

**Example input / output:**

| Input (Tagging) | Output (Updated Metadata) |
|-----------------|----------------------------|
| Model ID: 12345, UniqueID: 67890, Owner: DataScienceTeamB | Tags: {"UniqueID": "67890", "Owner": "DataScienceTeamB"} |

**Data:**

```json
{
  "modelId": "12345",
  "ownerTeam": "DataScienceTeamA",
  "tags": {
    "UniqueID": "12345",
    "Owner": "DataScienceTeamA"
  }
}
```

**Flow:**

1. Model/Product Creation
2. Tagging with Unique ID
3. Storing Metadata in ExternalDatabase
4. Defining Cost Attribution Rules
5. Real-Time Monitoring &Reporting
6. Team Split/Merge
7. Backfill Historical Costs
8. Update Cost Allocation

