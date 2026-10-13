---
type: "practice"
difficulty: "Deep"
---

# Given a Bookings table write a subquery-based query to find the customer who made the most bookings.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Subquery Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to identify the customer with the highest number of bookings from a Bookings table using a subquery. The core idea is to count the bookings per customer and then use that information to filter out the customer(s) who have the maximum booking count.

#### Triaging the Question

The question asks us to find the customer with the most bookings, which involves:

Counting the number of bookings for each customer.
Finding the highest booking count.
Identifying all customers who match this highest booking count.

We will assume that the Bookings table has columns like customer_id, booking_date, and possibly other details such as booking_amount.

#### First-Principles Solution

To achieve this, we can use a subquery to first determine the maximum number of bookings any customer made. Then, we use a JOIN or WHERE clause in the outer query to find all customers who have that maximum booking count.

Here is how you can write the SQL query:

SQL
WITH MaxBookings AS (
    SELECT MAX(count) as max_count
    FROM (
        SELECT COUNT(*) as count
        FROM Bookings
        GROUP BY customer_id
    ) AS BookingCounts
)
SELECT customer_id, COUNT(*)
FROM Bookings
GROUP BY customer_id
HAVING COUNT(*) = (SELECT max_count FROM MaxBookings);

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: This query uses a common table expression (CTE) to first find the maximum booking count across all customers. Then, it groups bookings by customer_id and filters out those who match this maximum count.

#### Common Mistakes
Not using a CTE or subquery to find the maximum booking count before filtering.
Incorrectly grouping or counting records in the main query.
Misunderstanding the requirement for finding all customers, not just one.
Common Follow-Ups
What if there are ties (multiple customers with the same highest number of bookings)?
How would you optimize this query for large datasets?
Can we modify the query to list all tied customers in a single result set?

**Code:**

```sql
WITH MaxBookings AS (
    SELECT MAX(count) as max_count
    FROM (
        SELECT COUNT(*) as count
        FROM Bookings
        GROUP BY customer_id
    ) AS BookingCounts
)
SELECT customer_id, COUNT(*)
FROM Bookings
GROUP BY customer_id
HAVING COUNT(*) = (SELECT max_count FROM MaxBookings);
```

