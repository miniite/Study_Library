---
type: "practice"
difficulty: "Intermediate"
---

# An interviewer challenges your use of ON DELETE CASCADE across three levels of hierarchy; what are the production risks and when would you switch to RESTRICT?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Modeling Relationships and Schemas](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database schema, especially one that involves multi-level hierarchies, ON DELETE CASCADE can be a powerful tool to maintain data integrity automatically. However, it comes with significant production risks, particularly in a three-level hierarchy scenario. In this context, we need to consider the trade-offs and when to opt for RESTRICT instead.

#### Triaging the Question

The interviewer is probing your understanding of:

Database Integrity: How deletions propagate through hierarchical data.
Risk Management: Understanding potential consequences that can arise from automatic cascading actions.
Choice of Constraints: When and why to use different types of referential integrity constraints like CASCADE vs. RESTRICT.

We assume the schema involves three levels:

Top-Level Entity (e.g., Departments)
Middle-Level Entity (e.g., Teams)
Bottom-Level Entity (e.g., Employees)

#### First-Principles Solution

Let’s break down the scenario with a simple example:

Schema:

PLAINTEXT
CREATE TABLE Department (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE Team (
    id INT PRIMARY KEY,
    department_id INT,
    name VARCHAR(100),
    FOREIGN KEY (department_id) REFERENCES Department(id) ON DELETE CASCADE
);

CREATE TABLE Employee (
    id INT PRIMARY KEY,
    team_id INT,
    name VARCHAR(100),
    FOREIGN KEY (team_id) REFERENCES Team(id) ON DELETE CASCADE
);

In this schema, ON DELETE CASCADE ensures that when a department or a team is deleted, all associated teams or employees are also deleted. However, using CASCADE at every level can lead to unintended data loss if not carefully managed.

Rationale and Risks
Scenario 1: Deleting a Department
Automatic Action: All Teams under the department will be deleted.
Risk: If some critical information is stored in these teams that should not be lost, cascading deletions could cause irreversible damage to data integrity.
Scenario 2: Deleting a Team
Automatic Action: All Employees in this team will be deleted.
Risk: Similarly, if specific employee records are crucial and should remain intact, CASCADE can lead to unintended loss of valuable data.
When to Use RESTRICT

To mitigate these risks, we need to consider using RESTRICT. Here’s how it works:

Modified Schema:

PLAINTEXT
CREATE TABLE Department (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE Team (
    id INT PRIMARY KEY,
    department_id INT,
    name VARCHAR(100),
    FOREIGN KEY (department_id) REFERENCES Department(id) ON DELETE RESTRICT
);

CREATE TABLE Employee (
    id INT PRIMARY KEY,
    team_id INT,
    name VARCHAR(100),
    FOREIGN KEY (team_id) REFERENCES Team(id) ON DELETE RESTRICT
);

With RESTRICT, no automatic deletion occurs. Instead, the operation will fail unless a proper backup or manual cleanup is done first.

Benefits of Using RESTRICT
Data Integrity: Ensures that no data can be accidentally deleted, maintaining integrity.
Controlled Data Management: Provides more control over when and how data is removed from the database.
Error Handling: Forces you to handle deletions in a structured manner, reducing the risk of cascading errors.

#### Closing Summary

In summary, ON DELETE CASCADE can simplify data management but comes with significant risks, particularly in hierarchical structures. Using RESTRICT offers better control and ensures data integrity by forcing explicit handling of deletions. The choice between CASCADE and RESTRICT depends on the specific requirements and potential consequences of automatic data removal.

#### Common Mistakes
and Follow-Ups
Overuse of CASCADE: Overusing CASCADE can lead to unintended data loss.
Lack of Understanding: Failing to understand the implications of cascading actions can result in serious production issues.
Manual vs. Automated: Always consider whether manual intervention is needed before allowing automatic deletions.

When designing such a schema, it's crucial to weigh these factors carefully and document your decisions explicitly.

**Code:**

```sql
CREATE TABLE Department (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE Team (
    id INT PRIMARY KEY,
    department_id INT,
    name VARCHAR(100),
    FOREIGN KEY (department_id) REFERENCES Department(id) ON DELETE RESTRICT
);

CREATE TABLE Employee (
    id INT PRIMARY KEY,
    team_id INT,
    name VARCHAR(100),
    FOREIGN KEY (team_id) REFERENCES Team(id) ON DELETE RESTRICT
);
```

