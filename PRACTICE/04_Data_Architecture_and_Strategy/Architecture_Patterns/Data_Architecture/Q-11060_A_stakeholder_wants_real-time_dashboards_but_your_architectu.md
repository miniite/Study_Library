---
type: "practice"
difficulty: "Intermediate"
---

# A stakeholder wants 'real-time dashboards' but your architecture is nightly batch. How do you figure out whether they genuinely need streaming, and what's the smallest change that gets them closer?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Streaming-first Data Architecture](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Streaming-first_Data_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a stakeholder who requests real-time dashboards in an architecture designed for nightly batch processing, we first need to clearly understand their needs and the current limitations of our architecture. The key is to explore whether they genuinely require streaming data or if there might be viable alternatives that could meet their requirements with minimal changes.

#### Triaging the Question

The core of this question revolves around:

Understanding the Stakeholder's Requirements: What specific real-time insights are needed? How critical is the timeliness of the data?
Evaluating Current Architecture Capabilities: Can nightly batch processing still provide sufficient and timely data for their needs?
Identifying Alternatives and Trade-offs: Are there simpler solutions that could offer a compromise between immediate insights and minimal changes to the existing architecture?

#### Assumptions
The current architecture is optimized for nightly batch processing, which may involve ETL pipelines with complex transformations.
Real-time dashboards imply near-instantaneous data updates, often requiring streaming technologies like Kafka or Apache Pulsar.
Step-by-Step Solution
1. Gather Requirements and Context

Start by having a detailed discussion with the stakeholder to understand their specific use case:

What are the business critical KPIs they want to monitor?
How often do these KPIs change? Is there a threshold where immediate data updates are crucial?
2. Assess Current Architecture’s Capabilities

Evaluate if nightly batch processing can still provide timely and relevant insights:

Determine the time gap between when new data is ingested and when it appears in dashboards.
Assess whether this delay meets the stakeholder's needs or if they need more frequent updates.
3. Explore Alternatives

#### Cons
ider simpler, less invasive solutions before jumping into a full streaming architecture:

Incremental Refreshes: Implement incremental refreshes within the nightly batch process to provide more up-to-date data without fully switching to real-time.
Nearline Storage and Processing: Use nearline storage solutions that can serve as a buffer between online dashboards and offline processing. This could involve setting up Apache Iceberg tables for faster queries on recent data.
4. Pilot and Prototype

Propose a small-scale pilot or prototype:

Develop a proof of concept (POC) to test the feasibility of implementing incremental refreshes.
Use this POC to demonstrate how nearline storage can improve dashboard performance without major architectural changes.

#### Closing Summary

By carefully assessing the stakeholder's requirements, understanding the limitations of our current architecture, and exploring alternative solutions, we can determine whether a full transition to real-time dashboards is necessary or if smaller, incremental improvements might suffice. This approach ensures that any changes made are both practical and aligned with business needs.

#### Common Mistakes/Follow-ups
Overlooking the potential for simpler, incremental improvements.
Failing to thoroughly understand the stakeholder's specific use case.
Assuming real-time data is always necessary without validating this assumption through detailed discussions and POCs.

**Comparison:**

```
Source	Target
DynamoDB	Redshift
```

**Code:**

```python
# Example of an incremental refresh function in a Python ETL pipeline
def incremental_refresh(source, target):
    # Query recent data from the source
    recent_data = query_recent_data(source)

    # Update the target with new data
    update_target(target, recent_data)
```

**Example input / output:**

```

Input:

Source Table: DynamoDB recent_changes
Target Database: Redshift live_metrics

Output:

Updated Redshift table with latest changes from DynamoDB
SQL
Copy
UPDATE live_metrics
SET metrics = (
  SELECT * FROM (SELECT * FROM recent_changes) AS new_data
)
WHERE ...; -- join condition to match updated data
```

**Data:**

```json
{
  "source": "DynamoDB",
  "target": "Redshift"
}
```

**Flow:**

1. Query Recent Data
2. Update Target
3. Refresh Live Dashboards

