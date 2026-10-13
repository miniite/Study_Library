---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute the Nth occurrence of an event per user (e.g. their 3rd purchase) and its timestamp using ROW_NUMBER or NTH_VALUE?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ First Last per Group and Running Totals](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/First_Last_per_Group_and_Running_Totals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of computing the (N)th occurrence of an event per user, such as their 3rd purchase, we can use SQL window functions like ROW_NUMBER() and NTH_VALUE(). The goal is to identify each user's specific (N)th event along with its timestamp. Here’s a step-by-step breakdown of how to achieve this:

#### Overview
We will first rank the events by some ordering criteria (e.g., purchase timestamp) for each user, then select the (N)th occurrence based on that ranking.

#### Triaging the Question
The interviewer is probing your understanding of window functions and how to apply them in a specific context. You need to make assumptions about the schema (e.g., table name, relevant columns), understand the requirement for (N), and consider edge cases such as when there are fewer than (N) events per user.

#### Solution

Start by ordering the purchases based on a timestamp or any other relevant criteria.
Use ROW_NUMBER() to assign a unique rank to each purchase within its user's partition of data.
Filter for the specific (N)th occurrence using the condition row_number = N.

Here’s how you can implement this in SQL:

SQL
SELECT
    user_id,
    purchase_timestamp,
    product_id,
    NTH_VALUE(product_id, 3) OVER (PARTITION BY user_id ORDER BY purchase_timestamp) AS third_product,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY purchase_timestamp) AS row_num
FROM
    purchases;

This query will return the user_id, purchase_timestamp, and product_id for each purchase, along with a column indicating the row number. The NTH_VALUE() function is used to get the 3rd product purchased by each user.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Ensure that you correctly partition the data by the relevant user identifier.
Use the correct ordering criteria (e.g., timestamp) to rank the events properly.
Be mindful of performance implications, especially with large datasets and high (N) values.

Potential follow-ups might include asking about handling ties in ranking, dealing with users who have fewer than (N) purchases, or optimizing the query for better performance.

**Comparison:**

| user_id | purchase_timestamp   | product_id | row_num |
|---------|----------------------|------------|---------|
| 1       | 2023-01-01 10:00     | A          | 1       |
| 1       | 2023-01-03 11:00     | B          | 2       |
| 1       | 2023-01-05 12:00     | C          | 3       |
| 2       | 2023-02-01 13:00     | D          | 1       |
| 2       | 2023-02-02 14:00     | E          | 2       |

**Code:**

```sql
SELECT
    user_id,
    purchase_timestamp,
    product_id,
    NTH_VALUE(product_id, 3) OVER (PARTITION BY user_id ORDER BY purchase_timestamp) AS third_product,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY purchase_timestamp) AS row_num
FROM
    purchases;
```

**Example input / output:**

```text
Input:
User purchases (sample data):
| user_id | purchase_timestamp   | product_id |
|---------|----------------------|------------|
| 1       | 2023-01-01 10:00     | A          |
| 1       | 2023-01-03 11:00     | B          |
| 1       | 2023-01-05 12:00     | C          |
| 2       | 2023-02-01 13:00     | D          |
| 2       | 2023-02-02 14:00     | E          |

Output:
| user_id | purchase_timestamp   | product_id | third_product |
|---------|----------------------|------------|---------------|
| 1       | 2023-01-05 12:00     | C          | C             |
```

**Data:**

```json
{
  "purchases": [
    {"user_id": 1, "purchase_timestamp": "2023-01-01 10:00", "product_id": "A"},
    {"user_id": 1, "purchase_timestamp": "2023-01-03 11:00", "product_id": "B"},
    {"user_id": 1, "purchase_timestamp": "2023-01-05 12:00", "product_id": "C"},
    {"user_id": 2, "purchase_timestamp": "2023-02-01 13:00", "product_id": "D"},
    {"user_id": 2, "purchase_timestamp": "2023-02-02 14:00", "product_id": "E"}
  ]
}
```

**Flow:**

1. Start
2. Partition by user_id
3. Order by purchase_timestamp
4. Assign row numbers usingROW_NUMBER()
5. Filter for Nth occurrence (N=3)
6. Output third product withtimestamp

