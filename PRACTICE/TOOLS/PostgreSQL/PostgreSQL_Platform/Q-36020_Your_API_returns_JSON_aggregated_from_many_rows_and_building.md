---
type: "practice"
difficulty: "Intermediate"
---

# Your API returns JSON aggregated from many rows and building it in Node is slow; when would you use jsonb_build_object/json_agg to shape the response in Postgres and what are the risks?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ FIRST_VALUE and LAST_VALUE](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/FIRST_VALUE_and_LAST_VALUE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize your API's response time, using jsonb_build_object and json_agg in PostgreSQL can be a powerful approach. These functions allow you to build complex JSON structures directly within the database, reducing the amount of work that needs to be done by your application server. However, there are several considerations and potential risks associated with this method.

The key idea is to leverage PostgreSQL's powerful query capabilities to generate the required JSON structure in a single database operation rather than doing it in Node.js or another backend language. This can significantly reduce latency and improve overall performance, especially when dealing with large datasets.

#### Triage
What the Question Is Asking

The question is probing your understanding of how to optimize data processing and JSON generation within PostgreSQL itself. It also aims to assess whether you are aware of potential trade-offs and risks associated with this approach.

#### Assumptions
You have a table that needs to be aggregated.
The aggregation logic is complex enough that it would benefit from being handled by PostgreSQL.
Your API framework can handle the resulting JSON output efficiently.

#### Solution
From First Principles
Step 1: Understanding jsonb_build_object and json_agg
jsonb_build_object: This function constructs a new jsonb object with key-value pairs from its arguments.
json_agg: This function aggregates multiple rows into a single JSON array, where each row is converted to JSON.
Step 2: Example Use Case

Suppose you have a transactions table that contains transaction data. You want to aggregate this data by user and return a response with each user's total transactions count and their most recent transaction details.

SQL
SELECT jsonb_build_object(
    'user_id', u.id,
    'total_transactions', COUNT(*),
    'most_recent_transaction', (
        SELECT jsonb_build_object(
            'id', t.id,
            'amount', t.amount,
            'timestamp', t.timestamp
        )
        FROM transactions t
        WHERE t.user_id = u.id
        ORDER BY t.timestamp DESC
        LIMIT 1
    )
) AS user_transactions
FROM users u
JOIN transactions t ON u.id = t.user_id
GROUP BY u.id;
Step 3: Risks and Trade-offs
Complexity: The query can become complex, especially when dealing with nested data structures.
Performance: While jsonb operations are optimized in PostgreSQL, a large number of rows or complex queries might still impact performance.
Maintainability: Hard-coded JSON generation logic within SQL queries can make the code harder to maintain and debug compared to application-side processing.

#### Closing Summary

Using jsonb_build_object and json_agg in PostgreSQL is an effective way to optimize your API's response time by reducing the load on your backend server. However, it’s crucial to weigh the benefits against potential complexities and performance impacts. Always consider profiling both approaches to determine which works best for your specific use case.

#### Common Mistakes/Follow-Ups
Failing to Consider Application Logic: Sometimes, application-side logic is necessary, especially if there are business rules or transformations that need to be applied.
Overusing SQL for Complex Data Structures: For very complex data structures, it might be more maintainable to use a backend language like Node.js.

**Comparison:**

```
has
USER
TRANSACTION
```

**Code:**

```sql
-- Example query using jsonb_build_object and json_agg
SELECT jsonb_build_object(
    'user_id', u.id,
    'total_transactions', COUNT(*),
    'most_recent_transaction', (
        SELECT jsonb_build_object(
            'id', t.id,
            'amount', t.amount,
            'timestamp', t.timestamp
        )
        FROM transactions t
        WHERE t.user_id = u.id
        ORDER BY t.timestamp DESC
        LIMIT 1
    )
) AS user_transactions
FROM users u
JOIN transactions t ON u.id = t.user_id
GROUP BY u.id;
```

**Example input / output:**

```text
Input:
- Users with transactions

Output:
[
  {"user_id": 1, "total_transactions": 2, "most_recent_transaction": {"id": 2, "amount": 20.0, "timestamp": "2023-10-02T09:00:00Z"}},
  {"user_id": 2, "total_transactions": 1, "most_recent_transaction": {"id": 3, "amount": 5.0, "timestamp": "2023-10-03T08:00:00Z"}}
]
```

**Data:**

```json
{
  "users": [
    { "id": 1 },
    { "id": 2 },
    { "id": 3 }
  ],
  "transactions": [
    { "user_id": 1, "amount": 10.5, "timestamp": "2023-10-01T09:00:00Z" },
    { "user_id": 1, "amount": 20.0, "timestamp": "2023-10-02T09:00:00Z" },
    { "user_id": 2, "amount": 5.0, "timestamp": "2023-10-03T08:00:00Z" }
  ]
}
```

**Flow:**

1. Start
2. Query Users and Transactions
3. Generate JSON withjsonb_build_object/json_agg
4. Return JSON Response
5. End

