---
type: "practice"
difficulty: "Intermediate"
---

# What happens if a fact arrives but its matching dimension row does not exist yet?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Special Dimension Types](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the scenario where a fact record arrives without its corresponding dimension in a star or snowflake schema, we need to understand how data modeling works and what potential issues could arise. This is a critical aspect of data warehousing and business intelligence systems. Let's break this down step-by-step.

#### Triaging the Question

The question essentially asks us to handle an edge case where a fact record references a dimension that doesn't exist in the database yet. We need to consider how our data model handles such situations, including whether we have robust mechanisms for inserting or updating dimensions and handling errors gracefully.

Key Concepts
Star Schema: Consists of one or more fact tables and multiple dimension tables.
Snowflake Schema: A variant of star schema where dimensions are normalized, leading to a hierarchical structure.
Data Integrity: Ensuring that all necessary data is present before inserting into the database.
Problem Breakdown
Fact Table: Contains numeric metrics (e.g., sales amount).
Dimension Table: Contains descriptive attributes (e.g., customer details, product information).

When we receive a fact record, it typically references specific dimension keys. If these dimensions do not exist in the database:

We need to decide whether to insert new rows into the dimension table.
We must handle potential errors or missing data.
Step-by-Step Solution
1. Check for Existing Dimension

First, we check if the dimension record exists by querying the dimension table using the provided key(s).

SQL
SELECT * FROM DIMENSION_TABLE WHERE KEY_COLUMN = 'expected_key';

If no rows are returned, proceed to handle the missing dimension.

2. Handle Missing Dimension (Insert or Update)

We have two options:

Insert New Row: Create a new row in the dimension table with the necessary information.
Update Existing Row: If there’s an existing row but it needs updates, update that record.
SQL
-- Inserting a new row
INSERT INTO DIMENSION_TABLE (KEY_COLUMN, COLUMN1, COLUMN2) VALUES ('expected_key', 'value1', 'value2');

-- Updating an existing row
UPDATE DIMENSION_TABLE SET COLUMN1 = 'new_value1', COLUMN2 = 'new_value2' WHERE KEY_COLUMN = 'expected_key';
3. Handle Fact Insertion

After handling the dimension, insert the fact record into the fact table.

SQL
INSERT INTO FACT_TABLE (DIMENSION_KEY, METRIC) VALUES ('expected_key', 100);

#### Example
Workflow

Let's consider an example where a sales transaction arrives and we need to process it:

Fact Table: SALES
Columns: CUSTOMER_ID, PRODUCT_ID, QUANTITY_SOLD
Dimension Table: CUSTOMERS
Columns: CUSTOMER_ID, NAME, EMAIL
Step-by-Step Workflow

Receive Sales Record: A new sales record arrives with CUSTOMER_ID = 'C001' and PRODUCT_ID = 'P005'.

Check for Customer:

SQL
SELECT * FROM CUSTOMERS WHERE CUSTOMER_ID = 'C001';

Handle Missing Customer:

No row found, so we insert the customer.
SQL
INSERT INTO CUSTOMERS (CUSTOMER_ID, NAME, EMAIL) VALUES ('C001', 'John Doe', 'john.doe@example.com');

Insert Sales Record:

SQL
INSERT INTO SALES (CUSTOMER_ID, PRODUCT_ID, QUANTITY_SOLD) VALUES ('C001', 'P005', 2);

#### Closing Summary
and Common Mistakes

In this scenario, handling missing dimension rows requires careful consideration of data integrity. The common pitfalls include:

Failing Silently: Not inserting or updating the dimension can lead to incorrect results.
Duplicate Entries: Inserting a new row for an existing key without updating it could result in duplicate records.

By following these steps and ensuring robust error handling, we can ensure that our data warehouse remains consistent and accurate.

Common Follow-Ups
What happens if multiple facts arrive with the same missing dimension?
How do you handle updates to dimensions after a fact has been inserted?

**Comparison:**

```
table CUSTOMERS {
    KEY: CUSTOMER_ID
    Columns: NAME, EMAIL
}
table SALES {
    KEY: CUSTOMER_ID, PRODUCT_ID
    Columns: QUANTITY_SOLD
}
```

**Code:**

```sql
-- Example SQL Queries

INSERT INTO DIMENSION_TABLE (KEY_COLUMN, COLUMN1, COLUMN2) VALUES ('expected_key', 'value1', 'value2');

UPDATE DIMENSION_TABLE SET COLUMN1 = 'new_value1', COLUMN2 = 'new_value2' WHERE KEY_COLUMN = 'expected_key';

INSERT INTO FACT_TABLE (DIMENSION_KEY, METRIC) VALUES ('expected_key', 100);
```

**Example input / output:**

```text
-- Checking for existing customer
SELECT * FROM CUSTOMERS WHERE CUSTOMER_ID = 'C001';

-- Inserting a new customer
INSERT INTO CUSTOMERS (CUSTOMER_ID, NAME, EMAIL) VALUES ('C001', 'John Doe', 'john.doe@example.com');

-- Inserting sales record
INSERT INTO SALES (CUSTOMER_ID, PRODUCT_ID, QUANTITY_SOLD) VALUES ('C001', 'P005', 2);
```

**Data:**

```json
{
  "SALES": {
    "columns": ["CUSTOMER_ID", "PRODUCT_ID", "QUANTITY_SOLD"],
    "data": [
      {"CUSTOMER_ID": "C001", "PRODUCT_ID": "P005", "QUANTITY_SOLD": 2}
    ]
  },
  "CUSTOMERS": {
    "columns": ["CUSTOMER_ID", "NAME", "EMAIL"],
    "data": []
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Sales Record
4. Check for Existing Customer
5. Proceed to Insert Fact
6. Insert New Customer
7. Insert Fact
8. Fact Inserted Successfully

