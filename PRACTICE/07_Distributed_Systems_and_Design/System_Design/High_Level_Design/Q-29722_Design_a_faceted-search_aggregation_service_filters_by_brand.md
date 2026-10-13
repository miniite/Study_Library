---
type: "practice"
difficulty: "Intermediate"
---

# Design a faceted-search/aggregation service (filters by brand price color) over a large index — how do you compute facet counts efficiently and keep them consistent with the filtered result set?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Backend Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Search_Backend_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient and consistent faceted-search/aggregation service, we need to focus on two main aspects: computing facet counts accurately and ensuring that these counts remain consistent as filters are applied. The core idea is to leverage a hybrid approach combining real-time updates with periodic batch processing. Here’s how we can achieve this:

TL;DR

We will use a combination of in-memory caching for quick lookups, database triggers for real-time updates, and scheduled batch jobs for consistency checks and optimizations.

#### Triaging the Question

The question is essentially asking about how to efficiently compute facet counts (like brand, price range, color) over a large index while maintaining data consistency. The interviewer likely wants you to consider both performance and accuracy in your design.

#### Assumptions
The service will be used for faceted search with filters like brand, price range, and color.
The dataset is large but updates are infrequent enough that real-time updates can handle most cases.
We need to support multiple filters simultaneously and compute counts dynamically based on the applied filters.

#### First Principles Solution

Real-Time Updates Using Triggers:

Database Triggers: Set up database triggers or use a query rewrite mechanism in your ORM layer to update facet counts whenever an insertion, deletion, or modification occurs.
Caching Layer: Use an in-memory caching system (like Redis) to store the current state of facet counts. This provides quick access to the latest counts.

Batch Jobs for Consistency Checks:

Schedule periodic batch jobs that run at off-peak times to perform consistency checks and optimizations.
These jobs can validate the data in the cache against the database, recompute necessary counts, and update any discrepancies found.

Hybrid Approach:

Combine real-time updates with consistent state checks to ensure high performance during regular operations while maintaining accuracy.
Detailed Steps

Database Schema:

Design a schema that stores documents along with their facets (brand, price range, color).
Implement indexes on the relevant fields for faster queries.

Real-Time Update Mechanism:

Use database triggers or ORM hooks to increment/decrement facet counts whenever there’s an update in the database.

#### Example
For a brand iPhone, every time we insert or delete a document related to iPhone, increment or decrement the count for that brand.

In-Memory Cache:

Store facet counts in an in-memory cache like Redis. This allows quick access and updates.

Batch Job Process:

Define batch jobs (using tools like Airflow, Luigi) to run at scheduled intervals (e.g., every hour).
These jobs will validate the data in the cache against the database and correct any discrepancies.

#### Closing Summary

By combining real-time updates with periodic batch job checks, we can achieve efficient and consistent facet counts. The key is to balance real-time performance with regular consistency checks, ensuring that our system remains robust even under heavy load or frequent changes.

#### Common Mistakes/Follow-Ups
Ignoring Consistency: Failing to run periodic checks can lead to stale data.
Over-Caching: Using too much cache without validation can cause data inconsistencies.
Scheduling Batch Jobs Too Often: Frequent batch jobs can introduce latency and overhead, whereas infrequent jobs might not catch issues in time.
Triaging

If the interviewer asks follow-ups, consider asking about specific trade-offs:

How often should we run the batch job?
What if there are multiple databases involved (e.g., microservices architecture)?
How do we handle distributed systems and consistency across services?

**Comparison:**

```
Facets	Values
Brands	iPhone, Samsung
Price Range	$100 - $200
Color	Red, Blue
```

**Code:**

```python
# Example ORM Hook in Python for real-time updates
from sqlalchemy import event

@event.listens_for(session, 'after_commit')
def update_facet_counts(session):
    # Update facet counts here based on changes made during the session
    pass
```

**Example input / output:**

```text
Input:
- Insert document with brand="iPhone" and color="Red"

Output:
- Facet count update for brands ({"brands": {"iPhone": 1}})
- In-memory cache updated
```

**Data:**

```json
{
  "facets": {
    "brands": ["iPhone", "Samsung"],
    "price_ranges": [100, 200],
    "colors": ["Red", "Blue"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Database Update
4. Is Real-Time Update?
5. Cached Facet Count Updated
6. Batch Job Triggered
7. In-Memory Cache Validated
8. Batch Job Runs at ScheduledInterval
9. Consistent Data in BothSystems

