---
type: "practice"
difficulty: "Intermediate"
---

# Your DELETE cascade unexpectedly removed rows in five child tables. How do you audit ON DELETE CASCADE chains before shipping such a change?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ DROP and CASCADE](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DROP_and_CASCADE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with database cascades, it's crucial to ensure that any changes you make don't inadvertently remove data from multiple interconnected tables. To audit ON DELETE CASCADE chains before making changes, you need a systematic approach that covers identifying the chain, understanding its impact, and implementing checks.

**Comparison:**

| Table    | Primary Key   | Foreign Keys  | Cascade Actions  |
|----------|---------------|---------------|------------------|
| customers| id            | -             | -                |
| orders   | order_id      | customer_id   | cascade          |
| items    | item_id       | order_id      | cascade          |
| payments | payment_id    | order_id      | cascade          |
| reviews  | review_id     | order_id      | cascade          |

**Code:**

```sql
-- Example of an ON DELETE CASCADE configuration
ALTER TABLE orders
ADD CONSTRAINT fk_order_customer
FOREIGN KEY (customer_id)
REFERENCES customers(id)
ON DELETE CASCADE;
```

**Example input / output:**

```
Example Input

Consider the orders table with a cascading delete to items, payments, and reviews. Before making any changes, we want to audit these chains.

SQL
Copy
-- Check current constraints
SELECT constraint_name FROM information_schema.constraint_column_usage
WHERE table_name = 'orders' AND referenced_table_name IS NOT NULL;
Example Output
constraint_name
fk_order_customer
fk_items_orders
fk_payments_orders
fk_reviews_orders
```

**Data:**

| Table    | Primary Key   | Foreign Keys  | Cascade Actions  |
|----------|---------------|---------------|------------------|
| customers| id            | -             | -                |
| orders   | order_id      | customer_id   | cascade          |
| items    | item_id       | order_id      | cascade          |
| payments | payment_id    | order_id      | cascade          |
| reviews  | review_id     | order_id      | cascade          |

**Flow:**

1. Yes
2. No
3. Start Audit
4. Are there cascadingconstraints?
5. Identify all tables in the chain
6. Map out the foreign keys andtheir actions
7. Check if any critical data isinvolved
8. Implement checks for potentialissues
9. Document changes andimpacts
10. No action needed
11. Perform a dry run of deletions
12. Review results for unintendedconsequences
13. Make necessary adjustments ifrequired

