---
type: "practice"
difficulty: "Deep"
---

# Walk me through modeling a hierarchical org dimension of arbitrary depth so cost rollups stay correct as reorgs happen mid-period across the enterprise.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Ragged Hierarchies](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Ragged_Hierarchies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When modeling an organizational hierarchy in a database, ensuring that cost rollups remain accurate even during organizational restructuring is critical. This involves understanding the nuances of star and snowflake schemas, especially when dealing with hierarchies of arbitrary depth.

In this scenario, we need to design a model where:

Hierarchies can be arbitrarily deep.
Cost data must be accurately rolled up at each level of the hierarchy.
Changes in organizational structure mid-period are handled without disrupting existing cost calculations.

#### Triage
What’s Being Probed

The interviewer is likely probing your understanding of hierarchical data modeling, particularly star and snowflake schemas. The core question revolves around ensuring accuracy during dynamic changes (reorgs) while maintaining historical integrity.

#### Assumptions
Hierarchical Data: Organizations can be nested at multiple levels.
Cost Rollup Requirement: Costs need to be accurately aggregated from leaf nodes up to root nodes.
Reorganization Dynamics: Changes in hierarchy can happen during a period and should not affect ongoing cost calculations.
Step-by-Step Solution

Define the Hierarchical Structure:

Use an entity-relationship diagram (ERD) to represent the organizational structure. Each employee can belong to one or more hierarchical levels.

Model Using Star Schema:

Create a Dimension table for the hierarchy, where each row represents a node in the hierarchy.
Include foreign keys linking employees to their respective nodes and potentially other relevant dimensions.

Aggregating Costs:

Use a fact table that captures cost data. Each record will link to an employee and include the specific cost type (e.g., expenses, salaries).

Maintaining Historical Integrity:

Implement time-stamping on changes to ensure that historical data remains immutable.
Use transactional integrity constraints like ON DELETE CASCADE or DEFERRABLE INITIALLY IMMEDIATE.

Handling Reorganizations:

During reorganization, update the hierarchy relationships without altering existing cost records.
Ensure that when an employee moves, their past costs are recalculated based on new hierarchy rules.

Rollup Calculation:

Implement a materialized view or use recursive SQL to ensure that cost rollups are always accurate and up-to-date.
Detailed Model

Dimension Table: Hierarchy

SQL
CREATE TABLE Hierarchy (
    id INT PRIMARY KEY,
    employee_id INT NOT NULL,
    parent_id INT DEFAULT 0, -- root node has no parent
    level INT,               -- depth in the hierarchy
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

Fact Table: Costs

SQL
CREATE TABLE Costs (
    id INT PRIMARY KEY,
    employee_id INT NOT NULL,
    cost_type VARCHAR(50) NOT NULL,
    amount DECIMAL(10, 2),
    transaction_date DATE NOT NULL,
    FOREIGN KEY (employee_id) REFERENCES Employees(id)
);

Rollup Calculation:

SQL
CREATE VIEW Cost_Rollups AS
SELECT
    h.id as hierarchy_id,
    SUM(c.amount) as total_cost
FROM
    Hierarchy h
JOIN
    Costs c ON h.employee_id = c.employee_id
GROUP BY
    h.id;
Edge Cases and Common Mistakes:
Dealing with Hierarchical Ambiguities: Ensure that each employee is uniquely placed in the hierarchy.

#### Cons
istency Across Reorganizations: Avoid issues where past data influences current calculations.

#### Performance Considerations
Materialized views or indexed hierarchies can help but may require careful tuning.

#### Closing Summary
and Follow-Ups

This model ensures that cost rollups remain accurate by leveraging a star schema to capture hierarchical relationships. Handling reorganizations requires careful updates to maintain historical integrity without disrupting ongoing calculations. Common pitfalls include improper handling of historical data and performance issues with large hierarchies or frequent changes.

**Comparison:**

| Hierarchy | Columns | Description |
|-----------|---------|-------------|
| id        | INT     | Primary key |
| employee_id | INT    | Employee ID |
| parent_id  | INT     | Parent ID (default 0 for root) |
| level      | INT     | Hierarchical depth |
| Costs | Columns | Description |
|-------|---------|-------------|
| id          | INT     | Primary key |
| employee_id | INT     | Employee ID |
| cost_type   | VARCHAR | Type of cost |
| amount      | DECIMAL | Cost amount  |
| transaction_date | DATE | Date of transaction |

**Code:**

```sql
CREATE TABLE Hierarchy (
    id INT PRIMARY KEY,
    employee_id INT NOT NULL,
    parent_id INT DEFAULT 0, -- root node has no parent
    level INT,              -- depth in the hierarchy
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE Costs (
    id INT PRIMARY KEY,
    employee_id INT NOT NULL,
    cost_type VARCHAR(50) NOT NULL,
    amount DECIMAL(10, 2),
    transaction_date DATE NOT NULL,
    FOREIGN KEY (employee_id) REFERENCES Employees(id)
);
```

