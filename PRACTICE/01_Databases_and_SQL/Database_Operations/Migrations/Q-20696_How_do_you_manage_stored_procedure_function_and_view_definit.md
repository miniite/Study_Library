---
type: "practice"
difficulty: "Intermediate"
---

# How do you manage stored procedure, function, and view definitions under version control alongside table migrations?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schema Migration Strategy](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Managing stored procedures, functions, and views along with table migrations is crucial to ensure data integrity and consistency across deployments. At a high level, this process involves using version-controlled scripts that are applied sequentially during database upgrades or rollbacks.

The core idea is to treat every change in the database schema, including DDL (data definition language) for tables, stored procedures, functions, and views, as part of the same version control system. This approach ensures that all changes are documented, tested, and can be rolled back if needed.

To start, let's define the problem: you need a robust strategy to manage these changes so that they align with table migrations, ensuring that your database remains in a consistent state after each deployment. The key steps involve:

Version Control System (VCS): Use a VCS like Git to track all schema and code changes.
Database Migration Tools: Leverage tools such as Liquibase or Flyway for managing database migrations.
Change Scripts: Write scripts that represent the changes you want to make, including DDL and DML statements.

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

Version control practices in a database context
The role of tools like Liquibase or Flyway
How to structure and manage schema changes effectively
Error handling and rollback strategies

#### Assumptions

You have basic knowledge of Git and version control.
You understand the importance of database migrations.

#### First-Principles Solution
1. Version Control System (VCS)
Use a VCS like Git to manage your schema and code changes. Each commit should represent a clear change, such as adding or updating a stored procedure.
SH
git init
git add .
git commit -m "Add new stored procedure for customer data processing"
2. Database Migration Tools
Use tools like Liquibase or Flyway to manage database migrations. These tools allow you to define changesets that can be applied in a controlled manner.
XML
<!-- Example of a Liquibase XML changeSet -->
<changeSet id="1" author="engineer">
    <sql>
        CREATE PROCEDURE getCustomerData (customerId INT) ...;
    </sql>
</changeSet>
3. Change Scripts
For each database change, create a corresponding script in your VCS. This script should be able to run on its own or as part of a series of migrations.
SQL
-- Example SQL for creating a view
CREATE VIEW customer_view AS SELECT * FROM customers;
4. Dependency Management
Ensure that dependencies between schema elements are managed properly. For instance, if a stored procedure depends on a specific table or another stored procedure, the changes to these dependent components should be applied in sequence.
SQL
-- Example dependency management
CREATE TABLE customers (id INT PRIMARY KEY);
-- Now create a view after the table is created
CREATE VIEW customer_view AS SELECT * FROM customers;
5. Automated Testing
Implement automated testing to validate that each migration works as expected before applying it in production.
SH
pytest test_schema.py
6. Rollback Strategy
Develop a rollback strategy for scenarios where migrations fail or need to be reversed due to issues.
XML
<!-- Example of a rollback changeSet -->
<changeSet id="rollback1" author="engineer">
    <sql>
        DROP VIEW customer_view;
    </sql>
</changeSet>

#### Closing Summary

By using a VCS, database migration tools, and structured change scripts, you can effectively manage stored procedures, functions, and views alongside table migrations. This approach ensures that your database schema remains consistent and testable across all environments.

#### Common Mistakes/Follow-Ups

Forgetting to include dependencies in the migration process.
Not having a robust rollback strategy in place.
Not testing changes adequately before deployment.

**Comparison:**

| Table                | Description              |
|----------------------|--------------------------|
| customers            | Stores customer details  |
| customer_view        | View for customer data   |

**Code:**

```xml
<!-- Example Liquibase ChangeSet -->
<changeSet id="1" author="engineer">
    <sql>
        CREATE PROCEDURE getCustomerData (customerId INT) ...;
    </sql>
</changeSet>

<!-- Example Rollback ChangeSet -->
<changeSet id="rollback1" author="engineer">
    <sql>
        DROP VIEW customer_view;
    </sql>
</changeSet>
```

**Example input / output:**

```
Input: git commit -m "Add new stored procedure"
Output: Commit message and confirmation from the VCS.
SH
Copy
$ git status
On branch main
Your branch is up to date with 'origin/main'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   schema.yml

no changes added to commit (use "git add" and/or "git commit -a")
```

**Data:**

```yaml
# Sample schema.yml for Flyway
resources:
  - sql/create_tables.sql
  - sql/create_views.sql
  - sql/create_procedures.sql
```

**Flow:**

1. Initialize VCS
2. Add schema changes
3. Commit changes
4. Apply migrations with tool
5. Automate testing
6. Raise rollback plan if necessary

