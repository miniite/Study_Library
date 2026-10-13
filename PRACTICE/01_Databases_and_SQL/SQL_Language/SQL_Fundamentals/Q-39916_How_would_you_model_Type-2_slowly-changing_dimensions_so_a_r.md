---
type: "practice"
difficulty: "Deep"
---

# How would you model Type-2 slowly-changing dimensions so a report reflects the price a customer had at transaction time? What keys and valid-from/valid-to columns do you add?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Joining Type 2 Dimensions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model Type-2 slowly-changing dimensions in SQL for accurately reflecting a customer's price at transaction time, we need to introduce two key columns into our dimension table: valid_from and valid_to. This approach ensures that historical pricing data is captured over time. Here’s how you would structure the solution:

#### Overview
We will modify the existing customer dimension table by adding valid_from and valid_to columns to track when a customer's record was active. When new transactions occur, we insert a new row with the updated details while keeping the old row as is.

Key Points:

Each transaction creates a new record in the dimension table.
The original record remains unchanged but marked as inactive (valid_to = current date).
The new record gets an active status (e.g., valid_from = transaction date, valid_to = NULL).

Step-by-Step Solution: a. Assumptions: Our dimension table contains columns like customer_id, name, email, etc. b. Adding Columns: Add the valid_from and valid_to columns to the customer dimension table. c. Inserting New Records: For each transaction, insert a new row with updated details and set appropriate dates for valid-from and valid-to.

#### Example
SQL:

SQL
-- Adding necessary columns to the customer dimension table
ALTER TABLE customers ADD COLUMN valid_from DATE;
ALTER TABLE customers ADD COLUMN valid_to DATE;

-- Sample transaction data
INSERT INTO customers (customer_id, name, email, valid_from, valid_to)
VALUES ('C001', 'Alice Johnson', 'alice@example.com', '2023-06-01', NULL);

Reasoning from First Principles:

By using Type-2 slowly-changing dimensions, we ensure that historical data is preserved and can be retrieved for reporting purposes.
This method allows us to track the lifecycle of each record, providing accurate context when generating reports.

#### Closing Summary
Adding valid_from and valid_to columns to your customer dimension table helps maintain a history of changes over time, ensuring that any report querying this data can accurately reflect the price at transaction time.

#### Common Mistakes/Follow-Ups

Failing to keep the original row active (i.e., not updating valid_to) might result in incorrect historical records.
Not handling the valid_from and valid_to columns correctly during transactions can lead to data inconsistencies.

