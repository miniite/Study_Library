---
type: "practice"
difficulty: "Deep"
---

# When ingesting CDC from a normalized OLTP source into a lakehouse how do you decide whether to rebuild denormalized dimensions with merge or with full reload?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Denormalization in Lakes](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to rebuild denormalized dimensions using a MERGE operation or a full RELOAD, we need to consider several factors, including data freshness requirements, performance impacts, and the specific use cases of our downstream analytics. In this scenario, we are dealing with Change Data Capture (CDC) from a normalized Online Transactional Processing (OLTP) source into a lakehouse environment.

The core concept here is understanding how MERGE and full reloads impact data consistency and the integrity of denormalized dimensions in a lakehouse architecture. MERGE allows for efficient updates to existing records, whereas full reloads might be more straightforward but can be resource-intensive.

**Comparison:**

| Customer ID | First Name | Last Name | Email                | Phone Number        |
|-------------|------------|-----------|----------------------|--------------------|
| 101         | Alice      | Johnson   | alice@example.com    | 555-4321           |
| 102         | Bob        | Smith     | null                 | null               |

**Code:**

```sql
-- Example SQL for a MERGE operation
MERGE INTO dim_customers AS target
USING (SELECT * FROM staging_customers) AS source
ON target.customer_id = source.customer_id
WHEN MATCHED THEN
    UPDATE SET target.first_name = source.first_name, target.last_name = source.last_name,
               target.email = source.email, target.phone_number = source.phone_number
WHEN NOT MATCHED THEN
    INSERT (customer_id, first_name, last_name, email, phone_number)
    VALUES (source.customer_id, source.first_name, source.last_name, source.email, source.phone_number);
```

**Example input / output:**

```text
Input (staging_customers):
- customer_id: 101, first_name: Alice, last_name: Johnson, email: alice@example.com, phone_number: 555-1234
- customer_id: 102, first_name: Bob, last_name: Smith, email: null, phone_number: null

Output (dim_customers after merge):
- customer_id: 101, first_name: Alice, last_name: Johnson, email: alice@example.com, phone_number: 555-4321
- customer_id: 102, first_name: Bob, last_name: Smith, email: null, phone_number: null

Input (staging_customers):
- customer_id: 101, first_name: Alice, last_name: Johnson, email: alice@example.com, phone_number: 555-1234
- customer_id: 102, first_name: Bob, last_name: Smith, email: null, phone_number: null

Output (dim_customers after merge):
- customer_id: 101, first_name: Alice, last_name: Johnson, email: alice@example.com, phone_number: 555-4321
- customer_id: 102, first_name: Bob, last_name: Smith, email: null, phone_number: null

Input (staging_customers):
- customer_id: 101, first_name: Alice, last_name: Johnson, email: alice@example.com, phone_number: 555-1234
- customer_id: 102, first_name: Bob, last_name: Smith, email: null, phone_number: null

Output (dim_customers after merge):
- customer_id: 101, first_name: Alice, last_name: Johnson, email: alice@example.com, phone_number: 555-4321
- customer_id: 102, first_name: Bob, last_name: Smith, email: null, phone_number: null

Input (staging_customers):
- customer_id: 101, first_name: Alice, last_name: Johnson, email: alice@example.com, phone_number: 555-1234
- customer_id: 102, first_name: Bob, last_name: Smith, email: null, phone_number: null

Output (dim_customers after merge):
- customer_id: 101, first_name: Alice, last_name: Johnson, email: alice@example.com, phone_number: 555-4321
- customer_id: 102, first_name: Bob, last_name: Smith, email: null, phone_number: null
```

**Data:**

```json
{
  "staging_customers": {
    "columns": ["customer_id", "first_name", "last_name", "email", "phone_number"],
    "rows": [
      {"customer_id": 101, "first_name": "Alice", "last_name": "Johnson", "email": "alice@example.com", "phone_number": "555-1234"},
      {"customer_id": 102, "first_name": "Bob", "last_name": "Smith", "email": null, "phone_number": null}
    ]
  },
  "dim_customers": {
    "columns": ["customer_id", "first_name", "last_name", "email", "phone_number"],
    "rows": [
      {"customer_id": 101, "first_name": "Alice", "last_name": "Johnson", "email": null, "phone_number": "555-4321"},
      {"customer_id": 102, "first_name": "Bob", "last_name": "Smith", "email": null, "phone_number": null}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Check CDC Records
5. Are there updates?
6. MERGE into Dim Customers
7. Full Reload Dim Customers
8. End

