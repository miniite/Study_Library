---
type: "practice"
difficulty: "Deep"
---

# When would you denormalize a computed/derived field into the item vs compute it on read? Give a case in a wide-column store where the write-time denormalization creates a consistency headache.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cassandra Replication and Read Path](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Cassandra_Replication_and_Read_Path.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When deciding between computing a derived field at write time versus reading and computing it on-the-fly, the key factors are consistency requirements, query patterns, and the trade-offs in performance and storage. In wide-column stores like Cassandra or DynamoDB, denormalizing fields can introduce data duplication but ensures strong read consistency. However, this must be balanced against the potential for increased complexity and write-time overhead.

#### Triaging the Question

The question is probing our understanding of when to use denormalization in a NoSQL database context, particularly focusing on wide-column stores like Cassandra or DynamoDB. We need to consider both read and write operations, along with consistency models and performance implications. Assumptions include familiarity with NoSQL concepts such as eventual consistency vs strong consistency.

Step-by-Step Solution
Understand the Consistency Models: In wide-column databases, we have a choice between eventual consistency (common in Cassandra) or strongly consistent reads (like DynamoDB). Each model has implications for how data is stored and queried.
Analyze Query Patterns:
High Read Frequency: If the field needs to be read frequently, denormalizing it can improve performance by reducing the number of queries needed.
Low Write Frequency: If writes are infrequent compared to reads, computing the field at write time and storing it can reduce computation overhead on read.
Evaluate Storage Overhead:
Denormalizing fields will increase storage requirements but may simplify the application logic.

#### Cons
ider Write Consistency Headaches: In a system where strong consistency is required, denormalizing could lead to versioning issues or stale data problems.
Case Study: Customer Activity Summary

Imagine an e-commerce application tracking customer activity summaries for analytics purposes. The goal is to provide a summary of recent activities (e.g., number of purchases, total spend) that can be displayed on the user's dashboard.

Write Time Denormalization in Cassandra
Scenario: A user makes a purchase.
Actions:
Update the Purchases column family with the new transaction.
Compute and store the summary (number of purchases, total spend) as derived fields within the same row.

#### Cons
istency Headache: If multiple writes are happening simultaneously, ensuring all updates to the summary are atomic is challenging. This can lead to inconsistent summaries if not handled properly.
Read Time Computation in DynamoDB
Scenario: Fetching a user's dashboard data.
Actions:
Query the Purchases table for recent transactions.
Compute the summary fields (number of purchases, total spend) on-the-fly based on the fetched data.
Advantages: No need to denormalize; simpler application logic.

#### Closing Summary

In a wide-column store like Cassandra or DynamoDB, denormalizing computed fields can improve read performance but introduces complexity and potential consistency issues. For scenarios with high read frequency and low write rate, denormalization is beneficial. However, for systems requiring strong consistency across writes, computing the field on-read might be the better choice to avoid data duplication and maintain integrity.

#### Common Mistakes/Follow-Ups
Over-Denormalizing: Storing too much redundant information can lead to increased storage costs and maintenance overhead.
Ignoring Query Patterns: Failing to consider how frequently fields are read can result in suboptimal design choices.

#### Cons
istency vs. Performance Trade-offs: Not understanding the implications of strong consistency models can lead to underperforming systems.

**Comparison:**

| Operation       | Cassandra Denormalize | DynamoDB Computation |
|-----------------|-----------------------|----------------------|
| Write Frequency | Low                   | High                 |
| Storage Overhead| Increased             | None                 |
| Read Complexity | Low                   | High                 |
| Consistency     | Potential Issues      | Strong               |

**Code:**

```python
# Example Python function for denormalizing a summary field on write
def update_customer_activity_summary(customer_id, transaction_amount):
    # Update purchases column family
    update_purchases(customer_id, transaction_amount)

    # Compute and store the summary fields
    num_purchases = get_num_purchases(customer_id)
    total_spend = sum_transaction_amounts(customer_id)
    set_summary_fields(customer_id, num_purchases, total_spend)

# Example function for reading the summary on-the-fly
def fetch_dashboard_data(customer_id):
    # Query purchases and compute summary fields
    transactions = query_purchases(customer_id)
    num_purchases = len(transactions)
    total_spend = sum([t['amount'] for t in transactions])

    return {
        'numPurchases': num_purchases,
        'totalSpend': total_spend
    }
```

**Example input / output:**

```text
Input:
- Customer makes a purchase of $200.5

Output (Cassandra):
- `purchases` table updated with new transaction.
- Summary fields (`numPurchases`, `totalSpend`) stored in the same row.

Input:
- Fetch dashboard data for customer 1

Output (DynamoDB):
- Query `purchases` table to get recent transactions.
- Compute summary fields based on fetched data: numPurchases=2, totalSpend=$350.8
```

**Data:**

```json
{
  "purchases": [
    {"customerId": 1, "transactionAmount": 200.5},
    {"customerId": 1, "transactionAmount": 150.3}
  ],
  "summaryFields": {
    "customerId": 1,
    "numPurchases": 2,
    "totalSpend": 350.8
  }
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Write Transaction
6. Is Write Frequency High?
7. Denormalize and StoreSummary
8. Compute Summary on Read
9. Frequent Reads
10. Store Summary Field?
11. Reduce Query Load
12. Compute On-Read

