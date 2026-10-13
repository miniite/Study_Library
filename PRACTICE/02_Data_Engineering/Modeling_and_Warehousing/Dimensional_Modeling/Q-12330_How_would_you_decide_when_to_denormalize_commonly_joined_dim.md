---
type: "practice"
difficulty: "Deep"
---

# How would you decide when to denormalize commonly joined dimension attributes directly onto the fact table, and what are the risks?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Aggregate Fact Tables](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Aggregate_Fact_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to denormalize commonly joined dimension attributes directly onto the fact table, we need to balance the benefits of faster query performance against potential drawbacks such as increased storage costs and data redundancy. The key idea here is that a well-considered denormalization can improve read performance significantly, but must be carefully weighed against long-term maintenance overhead.

First, let's define what "denormalize" means in this context: it involves adding columns from dimension tables directly to the fact table, which can reduce the number of joins required during query execution. However, this approach comes with trade-offs that need to be managed effectively.

#### Triaging the Question

The core issue is about making an informed decision on whether denormalizing certain attributes from dimensions into a fact table will provide better performance gains without incurring unacceptable costs or complexities. The interviewer is probing your understanding of data modeling principles, the impact of joins vs. denormalization, and how to balance these factors.

#### Assumptions
We assume that the primary use case involves frequent joins between dimension tables and the fact table.
The goal is to optimize read performance without significantly increasing write operations or compromising data integrity.
There are no specific constraints on storage costs, but they are a concern.

#### First-Principles Solution

Identify Common Joins: Start by identifying which dimension attributes are frequently used in joins with the fact table. Use query logs and performance monitoring tools to identify these high-frequency queries.

Analyze Data Cardinality: Evaluate the cardinality of the dimensions involved. Attributes from higher-cardinality dimensions (those with many unique values) are less suitable for denormalization due to increased storage costs and potential data redundancy issues.

#### Cons
ider Query Patterns: Understand how the dimension attributes are used in queries. For example, if an attribute is always used as a filter or aggregate, it might be beneficial to include it directly.

Evaluate Performance Impact: Use a controlled environment (e.g., a test database with similar data) to simulate denormalization and measure its impact on query performance. This can involve creating benchmarks where you compare the execution time of queries before and after denormalization.

Storage Cost Analysis: Calculate the additional storage required for each potential denormalized attribute. Compare this cost against the savings in join operations and overall read performance improvements.

Data Integrity Considerations: Ensure that any denormalization does not compromise data integrity. For example, if a dimension attribute is frequently updated, it might be better to keep it normalized.

Scalability and Maintainability: Consider how denormalization will affect future scalability and maintenance. Frequent updates or additions of attributes can become cumbersome in a denormalized structure.

Incremental Approach: If the decision is made to denormalize, consider starting with a small set of carefully chosen attributes rather than a wholesale redesign. This allows for iterative improvements based on real-world performance data.

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake 1: Overdenormalization - Denormalizing too many or the wrong attributes can lead to excessive storage costs without significant performance gains.
Mistake 2: Ignoring Query Patterns - Failing to analyze which queries are most impacted by joins can result in poor decisions about what to denormalize.

#### Follow-Up
The interviewer might ask you to discuss how you handle incremental changes or versioning of data models, especially when dealing with evolving schemas.

#### Example Scenario

Suppose a retail application frequently joins customer demographics (e.g., age group) with sales data. If the query performance is suboptimal due to these frequent joins, you might consider denormalizing the age group attribute into the fact table. However, if the product dimensions have high cardinality and are not often queried directly, they should remain normalized.

**Comparison:**

```
has
SALES_FACT
int
sales_amount
int
age_group
CUSTOMERS
int
id
PK
int
age
string
other_details
customer_id
```

**Code:**

```sql
-- Example of a denormalized query with joined attributes
SELECT SUM(sales_amount), customer_age_group
FROM sales_fact sf
JOIN customers c ON sf.customer_id = c.id
WHERE date BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY customer_age_group;
```

**Example input / output:**

```text
Before denormalization:
Query: SELECT SUM(sales_amount) FROM sales_fact sf JOIN customers c ON sf.customer_id = c.id WHERE date BETWEEN '2023-01-01' AND '2023-12-31';

After denormalization:
Query: SELECT SUM(sales_amount), customer_age_group FROM sales_fact sf WHERE date BETWEEN '2023-01-01' AND '2023-12-31';
```

**Data:**

```json
{
  "factTable": {
    "columns": [
      "sales_amount",
      "customer_id",
      "age_group"
    ]
  },
  "dimensionTables": [
    {
      "name": "customers",
      "columns": [
        "id",
        "age",
        "other_details"
      ]
    }
  ]
}
```

**Flow:**

1. Identify Common Joins
2. Analyze Data Cardinality
3. Consider Query Patterns
4. Evaluate Performance Impact
5. Storage Cost Analysis
6. Ensure Data Integrity
7. Scalability and Maintainability
8. Incremental Approach

