---
type: "practice"
difficulty: "Deep"
---

# How do you decide when a caching problem is really a data-modeling problem, and lead the org to denormalize or add a read model instead of piling on more cache layers? How do you make that case to teams invested in caching?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-layer Caching Architecture](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/Multi-layer_Caching_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Deciding whether a caching problem is fundamentally a data-modeling issue involves understanding the core business requirements and system performance bottlenecks. When you identify that data freshness, consistency, or query complexity is causing issues, it often points towards the need to restructure your database schema or introduce read models. This solution requires a strategic approach, balancing technical debt with long-term benefits.

#### Triaging the Question

The question revolves around distinguishing between caching and data-modeling challenges. The core issue at hand is whether investing more in cache layers is justified, versus addressing underlying data structure inefficiencies through denormalization or creating read models. This decision involves evaluating performance bottlenecks, consistency requirements, and the trade-offs between short-term fixes and long-term architecture improvements.

#### Assumptions
Performance Bottleneck: The system's performance issues are primarily driven by database access times.
Data Freshness Requirements: There is a clear understanding of how data freshness impacts business operations.
Scalability Concerns: The system must scale to handle increasing load without compromising performance.

#### First-Principles Solution
Step 1: Identify the Performance Bottleneck

First, identify if the performance bottleneck is due to database read times. Common signs include frequent or complex queries, high query response times, and a large number of database operations per request.

Yes
No
Performance Metrics
Bottleneck?
Identify Query Performance
Optimize Queries
Data Freshness?
Step 2: Evaluate Data Freshness and Consistency

If optimizing queries does not significantly improve performance, consider the data freshness requirements. Data staleness can impact decision-making processes, user experience, or compliance. Use metrics like Query Latency, Response Time, and Frequency of Updates to assess.

Yes
No
Data Freshness?
Check for Staleness
Implement Caching
Monitor Performance Impact
Denormalize or Add ReadModel
Step 3: Denormalize the Database Schema

If data freshness is not a critical issue, denormalizing the database schema can significantly improve read performance. This involves storing redundant data to reduce query complexity and latency.

Yes
No
Denormalization
Reduce Query Complexity
Improve Read Performance
Add Read Model
Step 4: Introduce a Read Model

If denormalization is not sufficient, consider adding a read model. A read model duplicates data from the primary database to optimize queries for read-heavy workloads.

Yes
No
Read Model
Optimize Queries
Improve Performance
Refine Data Schema
Step 5: Communicating with Teams

To convince teams invested in caching, present a case based on performance data and future scalability needs. Highlight the long-term benefits of addressing underlying issues over short-term fixes.

#### Closing Summary

Deciding between caching and data-modeling involves evaluating the root cause of performance bottlenecks and understanding business requirements for data freshness and consistency. By strategically denormalizing or introducing read models, you can address these issues more effectively than just layering on additional cache layers.

#### Common Mistakes/Follow-Ups
Ignoring Data Freshness: Failing to consider how data staleness impacts the system.
Over-Caching: Using caching without fully understanding query performance and data usage patterns.
Underestimating Database Changes: Thinking that minor schema changes will not have a significant impact.
Triaging Follow-Up Questions:
How do you measure database read times?
What are some common pitfalls when implementing denormalization?
Can you provide an example of how adding a read model improved performance in a similar system?

**Comparison:**

```
Query ID	Before Optimization (ms)	After Optimization (ms)
1	250	150
2	300	180
```

**Code:**

```python
# Example code to measure query latency
import time

def measure_query_latency(query):
    start_time = time.time()
    # Execute the database query here
    result = execute_query(query)
    end_time = time.time()
    return (end_time - start_time) * 1000  # in milliseconds
```

**Example input / output:**

```text
Input:
- Query to measure latency

Output:
- Latency in milliseconds before and after optimization
```

**Data:**

```json
{
  "query_latency": {
    "before_optimization": [
      {"query_id": 1, "latency_ms": 250},
      {"query_id": 2, "latency_ms": 300}
    ],
    "after_optimization": [
      {"query_id": 1, "latency_ms": 150},
      {"query_id": 2, "latency_ms": 180}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Performance Metrics
6. Bottleneck?
7. Identify Query Performance
8. Optimize Queries
9. Data Freshness?
10. Implement Caching
11. Monitor Performance Impact
12. Denormalize or Add ReadModel

