---
type: "practice"
difficulty: "Deep"
---

# Walk me through how you would decide between Spanner and CockroachDB for a new globally-distributed financial ledger, and what org-level factors (cloud lock-in, TrueTime vs HLC, ops maturity) dominate that decision.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ NewSQL Evaluation and Resilience](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/NewSQL_Evaluation_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between Google's Spanner and Cockroach Labs' CockroachDB for a new globally-distributed financial ledger, we must consider several critical factors including performance, availability, consistency, cloud lock-in, TrueTime vs HyperLogLogs (HLC), and operational maturity. Here’s how you might approach the decision from first principles.

To start, let's break down what Spanner and CockroachDB offer:

Spanner is a distributed relational database with eventual and linearizable consistency models, designed for global applications. It integrates TrueTime to manage distributed transactions across multiple data centers.
CockroachDB also supports strong consistency via HyperLogLogs (HLC) and is highly available but does not natively support the concept of TrueTime. Instead, it offers built-in recovery mechanisms that ensure data integrity.

#### Triaging the Question

The core of this question involves evaluating two key aspects:

Technical Suitability: How well each system meets the requirements for a globally-distributed financial ledger in terms of consistency and availability.
Operational and Organizational Factors: Cloud lock-in, maturity of operational support, and other organizational considerations.

#### First-Principles Solution
1. Technical Suitability

Spanner

#### Cons
istency Model: Spanner provides both eventual and linearizable consistency models. The ability to use TrueTime is crucial for financial transactions that require precise time-based decisions.
Availability: Spanner ensures high availability by maintaining consistent replication across multiple zones in different regions.

CockroachDB

#### Cons
istency Model: CockroachDB uses HyperLogLogs (HLC) which, while not as granular as TrueTime, still provides strong consistency and linearizability.
Availability: Similarly, CockroachDB ensures high availability through its built-in replication mechanisms.
2. Organizational Factors

Cloud Lock-In

Spanner is a managed service on Google Cloud, meaning you are locked into the Google ecosystem for managing and scaling your database. This could be beneficial if other Google services are already in use or required.
CockroachDB can run on-premises, AWS, GCP, or Azure, providing greater flexibility in terms of vendor lock-in.

TrueTime vs HLC

TrueTime is a powerful feature for financial applications where precise time management is critical. Spanner's implementation ensures that transactions are aware of the current time and can make decisions based on it.
HLCs (HyperLogLogs) are robust but may not be as granular or flexible in managing transactional time constraints compared to TrueTime.

Operational Maturity

Spanner has a higher level of operational maturity due to its integration with other Google services, extensive documentation, and active support from Google.
CockroachDB is well-documented and supported by the community. Its open-source nature means there are many resources available for troubleshooting and optimization.
3. Conclusion

The decision between Spanner and CockroachDB should be guided by a balanced approach:

If your organization prioritizes true temporal consistency, TrueTime from Spanner might be the better choice.
If flexibility in cloud deployment is more important, or if you prefer an open-source solution with community support, CockroachDB could be preferable.

#### Common Mistakes
and Follow-Ups
Overlooking Cloud Lock-In: Failing to consider long-term dependencies on a specific cloud provider can lead to unexpected costs and operational challenges.
Misunderstanding TrueTime vs HLC: Confusing the capabilities of both systems regarding time management could result in suboptimal transaction outcomes.
Neglecting Community Support: Ignoring the community support and documentation available for CockroachDB might overlook valuable resources that can enhance reliability and performance.

#### Closing Summary

Deciding between Spanner and CockroachDB requires a thorough evaluation of technical requirements, organizational factors, and long-term strategic goals. While both systems offer robust solutions for globally-distributed financial ledgers, the choice should align with your organization’s specific needs regarding consistency models, cloud lock-in, operational maturity, and support.

**Comparison:**

| Node | Region |
|------|--------|
| 1    | US     |
| 2    | EU     |
| 3    | APAC   |

**Code:**

```sql
-- Example CockroachDB Schema
CREATE TABLE transactions (
    id UUID PRIMARY KEY,
    amount DECIMAL(20, 6),
    description TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
) WITH (replication_factor=3);
```

**Example input / output:**

```text
Input:
- Transaction: {id: "abc123", amount: 100.50, description: "Payment to vendor"}

Output:
Transaction committed with id abc123 at US region on 2023-10-05T14:30:00Z
```

**Data:**

```json
{
  "ledger": {
    "nodes": [
      {"id": 1, "region": "US"},
      {"id": 2, "region": "EU"},
      {"id": 3, "region": "APAC"}
    ],
    "connections": [
      {"node_id": 1, "peer_node_id": 2},
      {"node_id": 2, "peer_node_id": 3}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Transaction Request
4. Is TrueTime Needed?
5. Use Spanner
6. Use CockroachDB
7. Transaction Committed withTrueTime
8. Transaction Committed withHLC

