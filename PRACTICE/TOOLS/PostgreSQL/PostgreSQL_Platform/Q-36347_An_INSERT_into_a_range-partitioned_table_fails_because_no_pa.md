---
type: "practice"
difficulty: "Deep"
---

# An INSERT into a range-partitioned table fails because no partition covers the value; how does a DEFAULT partition catch it, why is a growing default partition dangerous, and how do you automate partition creation?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Safe Migrations on Rolling Deploys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how a DEFAULT partition catches failed inserts in PostgreSQL's range-partitioned tables, we need to first grasp the concept of range partitioning, the role of default partitions, and the risks associated with them. Then, I'll detail an automated approach for creating new partitions.

#### Triaging the Question

The question revolves around a scenario where an INSERT fails because no specific partition matches the value being inserted. A DEFAULT partition is used as a catch-all to prevent data loss. However, it also presents risks if not managed properly, particularly when the default partition grows indefinitely. We need to explore these scenarios and then discuss how to automate partition creation.

#### First-Principles Solution
Understanding Range Partitioning

Range partitioning in PostgreSQL divides a table into sub-tables based on ranges of values for one or more columns. For instance:

SQL
CREATE TABLE sales (
    id SERIAL,
    region_id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount NUMERIC(10, 2)
) PARTITION BY RANGE (region_id, sale_date);
Role of Default Partitions

A DEFAULT partition acts as a catch-all for data that does not fit into any existing partitions. It ensures no data is lost when an insert fails due to lack of matching partition criteria.

SQL
CREATE TABLE sales_default PARTITION OF sales;

When you try to insert a value that doesn't match any existing partition's range, the DEFAULT partition will catch it:

SQL
INSERT INTO sales (region_id, sale_date, amount) VALUES (999, '2023-10-01', 150.00);
Risks of Growing Default Partitions

A growing default partition is dangerous because:

Performance: As the DEFAULT partition grows, it can degrade performance as the database has to perform extra work in managing and querying this larger table.
Maintenance: Managing a large DEFAULT partition manually becomes cumbersome.
Resource Usage: The storage and CPU resources consumed by a growing default partition are not optimized.
Automating Partition Creation

To avoid these issues, we can automate the creation of new partitions based on historical data distribution or specific criteria. This involves setting up rules for automatic partitioning.

Step-by-Step Solution
Identify Data Distribution: Use statistics to understand how data is distributed across existing partitions.
Define Criteria for New Partitions: Define a policy for when and how new partitions should be created, e.g., based on the number of rows in existing partitions or over a certain period.
Implement Automation:
Use PostgreSQL's LIST partitioning to define specific ranges.
Implement a custom script or trigger that checks conditions and creates new partitions as needed.

#### Example
Implementation
SQL
-- Create initial partitions based on region_id
CREATE TABLE sales_region_1 PARTITION OF sales FOR VALUES IN (1);
CREATE TABLE sales_region_2 PARTITION OF sales FOR VALUES IN (2);

-- Define a function to create new partitions
CREATE OR REPLACE FUNCTION create_new_sales_partition() RETURNS TRIGGER AS $$
BEGIN
    IF (SELECT COUNT(*) FROM sales WHERE region_id > 9) >= 500 THEN
        EXECUTE format('CREATE TABLE sales_region_%d PARTITION OF sales FOR VALUES IN (%d)',
                       greatest((SELECT max(region_id) FROM sales), 9)+1,
                       greatest((SELECT max(region_id) FROM sales), 9)+1);
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Create a trigger to call the function
CREATE TRIGGER partition_sales
AFTER INSERT ON sales FOR EACH STATEMENT EXECUTE FUNCTION create_new_sales_partition();

#### Closing Summary
and Common Mistakes
Summary: Automating partition creation can prevent default partitions from growing out of control, ensuring efficient data management.

#### Common Mistakes
Overusing DEFAULT partitions without proper automation.
Failing to monitor the growth of default partitions.
Not implementing a balanced approach between static and dynamic partitioning.

Automating partition creation is crucial for maintaining database performance and scalability. Regular monitoring and adjustment are essential to ensure that the system remains efficient and responsive.

**Comparison:**

```
region_id	sale_date	amount
1	2023-01-01	150.00
2	2023-02-01	200.00
3	2023-03-01	250.00
region_id	sale_date	amount
999	2023-10-01	150.00
```

**Code:**

```sql
CREATE TABLE sales (
    id SERIAL,
    region_id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount NUMERIC(10, 2)
) PARTITION BY RANGE (region_id, sale_date);

-- Default partition
CREATE TABLE sales_default PARTITION OF sales;

-- Function to create new partitions
CREATE OR REPLACE FUNCTION create_new_sales_partition() RETURNS TRIGGER AS $$
BEGIN
    IF (SELECT COUNT(*) FROM sales WHERE region_id > 9) >= 500 THEN
        EXECUTE format('CREATE TABLE sales_region_%d PARTITION OF sales FOR VALUES IN (%d)',
                       greatest((SELECT max(region_id) FROM sales), 9)+1,
                       greatest((SELECT max(region_id) FROM sales), 9)+1);
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Trigger to call the function
CREATE TRIGGER partition_sales
AFTER INSERT ON sales FOR EACH STATEMENT EXECUTE FUNCTION create_new_sales_partition();
```

**Example input / output:**

```text
-- Inserting data that fits in existing partitions
INSERT INTO sales (region_id, sale_date, amount) VALUES (1, '2023-01-01', 150.00);
-- Attempt to insert a value that does not fit any current partition
INSERT INTO sales (region_id, sale_date, amount) VALUES (999, '2023-10-01', 150.00);

-- After running the automation function
SELECT * FROM sales_region_1;
SELECT * FROM sales_default;
```

**Data:**

```json
{
  "region_id": [1, 2, 3],
  "sale_date": ["2023-01-01", "2023-02-01", "2023-03-01"],
  "amount": [150.00, 200.00, 250.00]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is data fitting into existingpartitions?
7. Insert data directly
8. Check if default partition exists
9. Use default partition for insert
10. Create new partition based oncriteria
11. Insert data and updatestatistics

