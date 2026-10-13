---
type: "practice"
difficulty: "Expert"
---

# How do you decide whether to adopt a polyglot persistence strategy versus consolidating, given each additional datastore adds on-call, backup, upgrade, and security surface?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choosing a Database for the Workload](<../../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between a polyglot persistence strategy (using multiple data stores) or consolidating into a single database type, there are several key factors to consider. A polyglot approach can offer flexibility and optimize performance, but it also introduces complexity and operational overhead. On the other hand, consolidating databases reduces complexity and may simplify operations. This decision is critical for ensuring both performance and cost efficiency while maintaining high availability and security.

To make an informed choice, we need to consider the following:

The nature of the data (e.g., relational vs. non-relational)
Query patterns and transactional requirements
Scalability needs and performance benchmarks
Compliance and regulatory considerations

We will break down each factor in detail using a first-principles approach.

#### Triaging the Question

The core of the question revolves around balancing flexibility, complexity, and operational overhead. We need to determine if the benefits of a polyglot strategy (e.g., handling different data types efficiently) outweigh the downsides of increased operational burden. The interviewer is likely probing your ability to weigh trade-offs and understand the implications on cost, maintenance, and security.

#### Assumptions
Data Types: We will consider both structured and unstructured data.
Query Patterns: Different systems may be optimized for specific types of queries (e.g., read-heavy vs. write-heavy).
Performance Requirements: High availability and latency requirements vary across applications.
Operational Constraints: Budget, time, and existing infrastructure are constraints.
Step-by-Step Solution

Data Analysis:

Identify the data types: structured (relational), semi-structured (JSON, XML), or unstructured (text, images).
Determine if there is a clear division of responsibilities among different data stores.

Query Patterns and Transactional Needs:

Analyze query patterns to understand read/write ratios.
Evaluate transactional requirements: ACID vs. eventual consistency.

Scalability and Performance:

Assess the scalability needs for each application.
Test performance benchmarks with different data stores.

Operational Considerations:

Estimate on-call time, backup policies, and upgrade cycles.
Evaluate security implications and compliance requirements.

Cost Analysis:

Calculate TCO (Total Cost of Ownership) for both strategies.
Compare licensing, operational costs, and potential savings.

#### Example
Calculation

Let’s assume we have three applications with the following characteristics:

App 1: High write operations, structured data, ACID transactions.
App 2: Low write, high read, semi-structured data, no strong consistency needs.
App 3: High write and read, unstructured data, real-time analysis.

We could consolidate these into a single RDBMS if the performance is sufficient. However, App 1 and App 2 may benefit more from separate databases: an RDBMS for App 1 and a NoSQL store for App 2. App 3 might require a NewSQL database to handle its unique requirements.

#### Closing Summary

In summary, deciding between polyglot persistence or consolidation involves thorough analysis of data types, query patterns, performance needs, and operational constraints. While polyglot strategies offer flexibility, they introduce complexity that must be carefully managed. A well-thought-out strategy can lead to a robust, scalable, and cost-effective solution.

#### Common Mistakes/Follow-Ups
Ignoring Operational Complexity: Overlooking the increased on-call time and operational burden.
Over-Engineering: Choosing too many data stores without clear benefit.
Neglecting Compliance: Failing to consider regulatory requirements.
Underestimating Costs: Not fully accounting for long-term TCO.

**Comparison:**

```
Application	Data Type	Query Pattern	Transactional Needs	Proposed Store
App 1	Structured	High Write	ACID	RDBMS
App 2	Semi-structured	Low Write, High Read	No strong consistency needs	NoSQL
```

**Code:**

```

-- Example SQL query for RDBMS consolidation strategy SELECT * FROM orders WHERE customer_id = 123 AND order_date >= '2023-01-01';

CODE
Copy

 ```javascript
// Example JavaScript code for NoSQL querying
const result = await db.collection('orders').find({ customerId: 123, orderDate: { $gte: new Date('2023-01-01') } }).toArray();
```

**Example input / output:**

```text
Input:
- App1: High write, structured data
- App2: Low write, high read, semi-structured

Output:
- Use RDBMS for App1 due to ACID requirements.
- Use NoSQL store for App2 due to query pattern.

Dry-run request response:
Request: "Choose database type for App1"
Response: "RDBMS - High write operations and structured data require ACID transactions."
```

**Data:**

```json
{
  "app1": {
    "type": "RDBMS",
    "queries": [
      {"operation": "INSERT", "ratio": 85},
      {"operation": "SELECT", "ratio": 15}
    ]
  },
  "app2": {
    "type": "NoSQL",
    "queries": [
      {"operation": "READ", "ratio": 90},
      {"operation": "INSERT", "ratio": 10}
    ]
  }
}
```

**Flow:**

1. Data Analysis
2. Relational Data
3. Non-relational Data
4. Semi-structured Data
5. Query Pattern Analysis
6. Read-Heavy, Write-Light
7. Choose RDBMS
8. Consolidate with RDBMS
9. Query Pattern Analysis
10. Write-Heavy, Read-Light
11. Choose NoSQL
12. Polyglot Strategy
13. Query Pattern Analysis
14. Real-Time Analytics
15. Choose NewSQL
16. Polyglot Strategy

