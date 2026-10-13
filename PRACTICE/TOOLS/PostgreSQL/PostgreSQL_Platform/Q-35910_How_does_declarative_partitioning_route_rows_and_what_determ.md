---
type: "practice"
difficulty: "Deep"
---

# How does declarative partitioning route rows, and what determines which partition an INSERT lands in?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Retention Purging at Scale](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Retention_Purging_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Declarative partitioning in PostgreSQL allows you to define a logical organization of data across tables or partitions without changing their physical layout. This means that you can still perform DDL operations on individual partitions while keeping the rest of your database structure simple and flat.

The core of declarative partitioning involves defining a partition key, which is used by the query planner to determine where a particular row should be inserted or queried from. In this question, we're specifically looking at how INSERT statements are routed to the correct partition based on their value for the partition key.

#### Triage
Understanding the Question

The interview question is probing your knowledge of PostgreSQL's declarative partitioning mechanism and how it interacts with the SQL insert operation. The key points to consider are:

How partition keys influence row routing.
The role of the query planner in deciding which partition to use.
Any constraints or rules that might affect this process.

#### Assumptions
We assume a basic understanding of PostgreSQL’s partitioning features, particularly list and range partitions.
The question assumes familiarity with how data is stored and accessed within these partitions.

#### First-Principles Solution
Partition Key and Data Routing

When you define a partition key in a PostgreSQL table, the system uses that key to determine which partition a row should belong to. For example:

SQL
CREATE TABLE sales (
    id SERIAL,
    product_id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount NUMERIC(10,2)
) PARTITION BY RANGE (sale_date);

In this case, sale_date is the partition key.

Range Partitioning

For range partitions, PostgreSQL will split the data based on specified ranges. For example:

SQL
CREATE TABLE sales_y2023 PARTITION OF sales FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

This creates a partition for all rows where sale_date is between '2023-01-01' and '2024-01-01'. The query planner then uses this information to decide which partition to insert the new row into.

List Partitioning

For list partitions, you can define specific lists of values for each partition. For example:

SQL
CREATE TABLE sales_by_product PARTITION OF sales FOR VALUES IN ('prodA', 'prodB');

Here, only rows where product_id is either 'prodA' or 'prodB' will be inserted into this partition.

Handling Unknown Values

If the value of the partition key is not known at the time of insertion and does not match any existing partitions, PostgreSQL will attempt to insert the row into a default partition if one has been defined. If no default partition exists, it will raise an error.

#### Example
Workflow

Let's consider an example where we are inserting data with sale_date set to '2023-10-15'. The relevant partitions and their ranges might look like this:

SQL
CREATE TABLE sales_y2022 PARTITION OF sales FOR VALUES FROM ('2022-01-01') TO ('2023-01-01');
CREATE TABLE sales_y2023 PARTITION OF sales FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

When you insert a row with sale_date = '2023-10-15', PostgreSQL will look at the partition key value and determine that this row belongs in sales_y2023. The query planner will then route the data to that specific partition.

#### Closing Summary

In summary, declarative partitioning in PostgreSQL uses the values of specified partition keys to route rows into appropriate partitions. This routing is handled by the query planner during an INSERT operation. Understanding how this works is crucial for optimizing your database performance and managing large datasets effectively.

#### Common Mistakes
and Follow-Ups
Incorrect Partition Key Selection: Using a poor partition key can lead to suboptimal query performance.
Overly Granular Partitions: Creating too many partitions can increase maintenance overhead.
No Default Partition: Failing to define a default partition can result in errors for out-of-range values.

**Code:**

```

CREATE TABLE sales ( id SERIAL, product_id INT NOT NULL, sale_date DATE NOT NULL, amount NUMERIC(10,2) ) PARTITION BY RANGE (sale_date);

CODE
Copy

```sql
CREATE TABLE sales_y2023 PARTITION OF sales FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
```

