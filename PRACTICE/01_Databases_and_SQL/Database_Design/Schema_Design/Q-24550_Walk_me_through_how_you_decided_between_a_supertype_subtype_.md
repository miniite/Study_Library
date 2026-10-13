---
type: "practice"
difficulty: "Deep"
---

# Walk me through how you decided between a supertype/subtype table-per-hierarchy versus table-per-type design for a polymorphic entity in production, and what queries suffered.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Surrogate vs Natural Keys for Lookups](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database schema for a polymorphic entity, choosing between a "supertype/subtype" (table-per-hierarchy) and "table-per-type" design involves balancing factors like query performance, data normalization, and maintenance complexity. In this scenario, let's consider a scenario where we need to model different types of users in an application—e.g., Employee, Customer, and Manager. We'll delve into the decision-making process from first principles.

**Code:**

```sql
-- Example schema for table-per-type design
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE Employees (
    user_id INT REFERENCES Users(id),
    department VARCHAR(100)
);

CREATE TABLE Customers (
    user_id INT REFERENCES Users(id),
    billing_address TEXT
);

CREATE TABLE Managers (
    user_id INT REFERENCES Users(id),
    reports_to INT REFERENCES Users(id)
);
```

**Example input / output:**

```text
No specific input-output examples required here, but we'll walk through a typical query scenario to illustrate the point.
```

**Flow:**

1. Yes
2. No
3. Complex joins
4. Simple queries
5. Low
6. High
7. Supertype/Subtype(Table-per-Hierarchy)
8. Is data normalized?
9. Minimized redundancy, easierto maintain
10. More complex queries, highermaintenance cost
11. Query Performance
12. Slower query performance
13. Fast query performance
14. Maintenance Complexity
15. Easier to manage
16. More complex schemachanges, harder to maintain

