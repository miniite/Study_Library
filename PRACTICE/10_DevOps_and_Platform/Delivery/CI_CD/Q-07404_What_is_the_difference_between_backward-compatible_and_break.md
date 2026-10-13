---
type: "practice"
difficulty: "Deep"
---

# What is the difference between backward-compatible and breaking database migrations in CD?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schema Changes in Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Schema_Changes_in_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Continuous Delivery (CD), both backward-compatible and breaking database migrations are essential practices, but they serve different purposes and have distinct impacts on the software lifecycle. Backward-compatible migrations ensure that new versions of a database schema can still work with old data and applications without issues, whereas breaking migrations require changes in existing code to handle new schema structures.

#### Triaging the Question

The core of this question is about understanding how backward-compatibility affects the migration process in CD environments. We need to explore the implications of these two types of migrations on application stability, development efficiency, and deployment strategies. Key assumptions include the use of SQL for database changes, version control systems like Git, and CI/CD pipelines.

#### First-Principles Solution
Backward-Compatible Migrations: These migrations are designed to maintain data integrity and compatibility across different versions of an application. For example, adding a new column with a default value or renaming a table without changing its structure.
Breaking Migrations: These involve significant changes in the database schema that may require modifications in the application codebase. For instance, removing a table, altering existing columns to change their data type, or splitting a single table into multiple tables.
Detailed Explanation
Backward-Compatible Migrations
Why Use Them: Ensures that older versions of an application can still work with new database schemas without breaking.

#### Example Scenario
Suppose you have a user profile table and need to add a new preferences field. Instead of just adding the column, which is backward-compatible, consider using a default value like 'default_preferences' or even a JSON column that allows flexibility in future modifications.
Impact on CI/CD Pipelines: Such migrations can be automated without requiring changes in application logic, reducing risk and improving deployment speed.
Breaking Migrations
Why Use Them: May be necessary for significant schema changes that cannot be achieved using backward-compatible methods. Examples include data restructuring to improve query performance or removing obsolete data models.

#### Example Scenario
If your current database structure has a single orders table with all order details, you might decide to split it into separate tables for order_headers, line_items, and payments. This requires changes in the application code to adapt to new structures.
Impact on CI/CD Pipelines: These migrations often require manual validation and potentially longer deployment windows due to the complexity of changing application logic.

#### Common Mistakes
and Follow-Ups
Overusing Breaking Migrations: Frequent breaking migrations can lead to increased development overhead, decreased stability, and frequent disruptions in CD pipelines.
Underestimating Backward-Compatible Solutions: Failing to leverage backward-compatible strategies can result in unnecessary workarounds that make code harder to maintain over time.
Ignoring Data Integrity: Both types of migrations must be designed with data integrity in mind; otherwise, they could lead to data loss or corruption.

#### Closing Summary

Backward-compatible and breaking database migrations serve distinct roles in CD environments. While backward-compatibility ensures smooth transitions without disrupting older applications, breaking migrations allow for more significant schema changes at the cost of increased complexity and potential instability. Understanding these differences helps in making informed decisions about when each approach is appropriate.

**Comparison:**

```
Migration Type	Description	Example Changes
Backward-Compatible	Maintains compatibility with older data/applications.	Adding columns with default values, renaming fields without changing their meaning.
Breaking	Requires changes in application logic to adapt.	Splitting tables, altering column types, removing obsolete fields.
```

**Code:**

```sql
-- Example of a backward-compatible migration (adding a new column)
ALTER TABLE users ADD COLUMN preferences JSONB DEFAULT '{}'::JSONB;

-- Example of a breaking migration (renaming and splitting tables)
RENAME TABLE orders TO order_headers;
CREATE TABLE line_items (
    id SERIAL PRIMARY KEY,
    order_header_id INT REFERENCES order_headers(id),
    product_name VARCHAR(255),
    quantity INT
);
CREATE TABLE payments (
    id SERIAL PRIMARY KEY,
    order_header_id INT REFERENCES order_headers(id),
    amount DECIMAL,
    date TIMESTAMP
);
```

**Example input / output:**

```text
Input:
- Current database schema with a single `orders` table.

Output:
- New database schema with separate tables for `order_headers`, `line_items`, and `payments`.
- Application code updated to use the new structure.
```

**Data:**

```json
{
  "user": {
    "id": 1,
    "preferences": {}
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Backward-CompatibleMigration
6. Is Data Integrity Maintained?
7. Automate and Deploy
8. Manual Validation Required
9. Breaking Migration
10. Does Code Need Changes?
11. Coordinate with Developmentfor Code Updates
12. Deploy Directly from DatabaseSchema Changes

