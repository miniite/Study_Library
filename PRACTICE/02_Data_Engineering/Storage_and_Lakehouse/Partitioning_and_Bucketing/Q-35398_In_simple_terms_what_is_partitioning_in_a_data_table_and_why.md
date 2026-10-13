---
type: "practice"
difficulty: "Basic"
---

# In simple terms, what is partitioning in a data table and why do we do it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Partitioning and Bucketing Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Partitioning in a data table is a method used to split large datasets into smaller, more manageable parts based on specific columns or values. This technique helps improve query performance by reducing the amount of data that needs to be scanned during execution. Essentially, it allows us to organize and manage our data in a way that aligns with how we query it, leading to faster and more efficient processing.

To understand why partitioning is useful, let's break down what it involves step by step from first principles.

**Code:**

```python
# Example of creating a partitioned table in Apache Hive (HQL)
CREATE TABLE sales (
  order_id INT,
  product_name STRING,
  price DECIMAL(10,2),
  sale_date DATE
)
PARTITIONED BY (year INT, month INT);
```

**Example input / output:**

```text
INSERT INTO sales VALUES
(1001, 'Laptop', 999.99, '2023-11-15'),
(1002, 'Monitor', 499.99, '2023-11-16');
```

**Flow:**

1. Yes
2. No
3. Data Table
4. Query Condition
5. Partition Column Check
6. Direct Access to Partition
7. Full Scan or Further Filtering
8. Efficient Query Execution

