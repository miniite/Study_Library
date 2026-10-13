---
type: "practice"
difficulty: "Deep"
---

# MySQL has no native materialized views. Describe a trigger-plus-summary-table approach and its correctness pitfalls.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized View Refresh and Maintenance](<../../../../01_Databases_and_SQL/Database_Objects/Materialized_Views/Materialized_View_Refresh_and_Maintenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

In MySQL, where native materialized view support is lacking, we can simulate this functionality using a combination of triggers and summary tables. This method involves creating a summary table that gets updated whenever the base table changes, ensuring that the summary always reflects the latest data.

#### Triage

The question asks us to describe a trigger-plus-summary-table approach for simulating materialized views in MySQL, along with potential correctness pitfalls. The core idea is to use triggers to keep a separate "summary" table up-to-date whenever there are changes in the base table. The interviewer might be probing your understanding of database triggers and their limitations.

#### First-Principles Solution
Define the Base Table: Assume we have a Sales table with columns like id, product_id, quantity_sold, and sale_date.
Create a Summary Table: Create a summary table, say SummarySales, which will keep track of aggregated data (e.g., total quantity sold by product).
Trigger for Data Changes: Write a trigger on the Sales table that updates the SummarySales table whenever there's an insert or update in the Sales table.
Handling Deletes and Triggers: Ensure deletes are properly accounted for, possibly using another set of triggers to handle such operations.
Step-by-Step Solution:

Base Table Schema:

SQL
CREATE TABLE Sales (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    quantity_sold INT NOT NULL,
    sale_date DATE NOT NULL
);

Summary Table Schema:

SQL
CREATE TABLE SummarySales (
    product_id INT PRIMARY KEY,
    total_quantity_sold BIGINT NOT NULL DEFAULT 0
);

Trigger for Inserts and Updates:

SQL
DELIMITER //
CREATE TRIGGER UpdateSummarySales AFTER INSERT ON Sales FOR EACH ROW
BEGIN
    UPDATE SummarySales
    SET total_quantity_sold = total_quantity_sold + NEW.quantity_sold
    WHERE product_id = NEW.product_id;

    IF NOT EXISTS (SELECT 1 FROM SummarySales WHERE product_id = NEW.product_id) THEN
        INSERT INTO SummarySales (product_id, total_quantity_sold)
        VALUES (NEW.product_id, NEW.quantity_sold);
    END IF;
END//
DELIMITER ;

-- Similar trigger for updates:
DELIMITER //
CREATE TRIGGER UpdateSummarySales AFTER UPDATE ON Sales FOR EACH ROW
BEGIN
    UPDATE SummarySales
    SET total_quantity_sold = total_quantity_sold - OLD.quantity_sold + NEW.quantity_sold
    WHERE product_id = NEW.product_id;

    IF NOT EXISTS (SELECT 1 FROM SummarySales WHERE product_id = NEW.product_id) THEN
        INSERT INTO SummarySales (product_id, total_quantity_sold)
        VALUES (NEW.product_id, NEW.quantity_sold);
    END IF;
END//
DELIMITER ;

Handling Deletes:

SQL
DELIMITER //
CREATE TRIGGER UpdateSummarySales AFTER DELETE ON Sales FOR EACH ROW
BEGIN
    UPDATE SummarySales
    SET total_quantity_sold = total_quantity_sold - OLD.quantity_sold
    WHERE product_id = OLD.product_id;

    IF NOT EXISTS (SELECT 1 FROM Sales WHERE product_id = OLD.product_id) THEN
        DELETE FROM SummarySales WHERE product_id = OLD.product_id;
    END IF;
END//
DELIMITER ;

#### Closing Summary
and Common Mistakes:

The key takeaway is that while this approach works, it has limitations. Specifically, the summary table may become stale if there are concurrent updates or deletes that aren't handled correctly in the trigger sequence. Additionally, handling complex queries involving multiple tables or more sophisticated aggregations can be challenging.

Common Pitfalls:
Concurrency Issues: Ensuring atomicity and consistency during concurrent transactions.
Stale Data: The summary table might become outdated if updates are not properly propagated through triggers.
Performance Overhead: Triggers can introduce significant overhead, especially with large datasets.
Complex Queries: Handling complex queries that require joins between the base and summary tables.

**Code:**

```

-- Base Table Schema CREATE TABLE Sales ( id INT AUTO_INCREMENT PRIMARY KEY, product_id INT NOT NULL, quantity_sold INT NOT NULL, sale_date DATE NOT NULL );

-- Summary Table Schema CREATE TABLE SummarySales ( product_id INT PRIMARY KEY, total_quantity_sold BIGINT NOT NULL DEFAULT 0 );

CODE
Copy

```sql
-- Trigger for Inserts and Updates
DELIMITER //
CREATE TRIGGER UpdateSummarySales AFTER INSERT ON Sales FOR EACH ROW
BEGIN
    UPDATE SummarySales
    SET total_quantity_sold = total_quantity_sold + NEW.quantity_sold
    WHERE product_id = NEW.product_id;

    IF NOT EXISTS (SELECT 1 FROM SummarySales WHERE product_id = NEW.product_id) THEN
        INSERT INTO SummarySales (product_id, total_quantity_sold)
        VALUES (NEW.product_id, NEW.quantity_sold);
    END IF;
END//
DELIMITER ;

-- Similar trigger for updates:
DELIMITER //
CREATE TRIGGER UpdateSummarySales AFTER UPDATE ON Sales FOR EACH ROW
BEGIN
    UPDATE SummarySales
    SET total_quantity_sold = total_quantity_sold - OLD.quantity_sold + NEW.quantity_sold
    WHERE product_id = NEW.product_id;

    IF NOT EXISTS (SELECT 1 FROM SummarySales WHERE product_id = NEW.product_id) THEN
        INSERT INTO SummarySales (product_id, total_quantity_sold)
        VALUES (NEW.product_id, NEW.quantity_sold);
    END IF;
END//
DELIMITER ;

-- Handling Deletes
DELIMITER //
CREATE TRIGGER UpdateSummarySales AFTER DELETE ON Sales FOR EACH ROW
BEGIN
    UPDATE SummarySales
    SET total_quantity_sold = total_quantity_sold - OLD.quantity_sold
    WHERE product_id = OLD.product_id;

    IF NOT EXISTS (SELECT 1 FROM Sales WHERE product_id = OLD.product_id) THEN
        DELETE FROM SummarySales WHERE product_id = OLD.product_id;
    END IF;
END//
DELIMITER ;
```

