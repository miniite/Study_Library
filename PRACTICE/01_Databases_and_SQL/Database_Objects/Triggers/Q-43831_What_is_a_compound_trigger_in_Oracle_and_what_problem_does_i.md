---
type: "practice"
difficulty: "Deep"
---

# What is a compound trigger in Oracle and what problem does it help solve?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DML vs DDL Triggers](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/DML_vs_DDL_Triggers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: A compound trigger in Oracle allows you to define multiple SQL statements within a single PL/SQL block, which can be used at different points of a DML (Data Manipulation Language) operation. This helps solve the problem of managing complex business logic that needs to be applied across various stages of data modification.

#### Triaging the Question

What is a Compound Trigger?: A compound trigger in Oracle is a special kind of PL/SQL block that combines multiple SQL statements and procedural code within one unit, allowing you to manage DML operations (INSERT, UPDATE, DELETE) more effectively.
Problem It Solves: The primary problem it addresses is the need for complex logic during data modification. By encapsulating both the procedural and SQL parts of a trigger in one place, compound triggers simplify the management of intricate business rules.

#### First Principles Solution

Understanding Triggers in Oracle: Before diving into compound triggers, it's essential to understand standard PL/SQL triggers. These are separate blocks for each DML event (INSERT, UPDATE, DELETE) and often contain both procedural logic and SQL commands.
Introducing Compound Triggers:
A compound trigger is defined with the FOR EACH ROW clause, just like a regular trigger.
It can include multiple sections: BEFORE STATEMENT, BEFORE EACH ROW, AFTER EACH ROW, and AFTER STATEMENT. This flexibility allows you to place different types of logic at various stages of the DML operation.

#### Example
SQL
CREATE OR REPLACE TRIGGER compound_trigger_example
FOR INSERT OR UPDATE OR DELETE ON employees
COMPOUND TRIGGER

  -- Global variables or initialization code can go here

  BEFORE STATEMENT IS
    BEGIN
      -- Code that runs before any rows are processed
    END;

  BEFORE EACH ROW IS
    BEGIN
      -- Code that runs for each row before modification
    END;

  AFTER EACH ROW IS
    BEGIN
      -- Code that runs after the current row is modified
    END;

  AFTER STATEMENT IS
    BEGIN
      -- Code that runs after all rows have been processed
    END;

END;
Common Usage Scenarios:
Logging Changes: You can log changes made by each DML operation.
Transaction Management: Handle complex transaction logic, such as rolling back or committing based on certain conditions.

#### Cons
traint Enforcement: Implement custom validation rules that span multiple rows.

#### Closing Summary
Compound triggers in Oracle provide a powerful mechanism to manage complex business logic during data modification operations. By allowing you to encapsulate both procedural and SQL statements within a single block, they simplify the implementation of intricate rules and enhance code readability and maintainability.

#### Common Mistakes/Follow-Ups

Misusing Compound Triggers: Overusing compound triggers can lead to overly complex and hard-to-maintain code. It’s important to weigh their complexity against simpler alternatives.
Ignoring Performance Considerations: While compound triggers offer flexibility, they should be used judiciously as they might impact performance due to the overhead of managing multiple statements within a single block.

