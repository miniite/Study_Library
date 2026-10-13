---
type: "practice"
difficulty: "Intermediate"
---

# What is a lookup or reference table and give an example of when to use one?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Polymorphic Associations](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Polymorphic_Associations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A lookup or reference table in database design is used to store static data that can be referenced by other tables. It helps maintain data integrity, ensures consistency, and reduces redundancy. For instance, a State table could contain all U.S. states with their abbreviations, which can be linked from the User table via a foreign key.

The question is probing your understanding of database normalization principles and how to effectively manage related entities by breaking down data into smaller, more manageable chunks while maintaining referential integrity. Let's explore this step-by-step from first principles.

**Comparison:**

```
ID	Name	Abbreviation
1	Alabama	AL
2	Alaska	AK
...	...	...
5	California	CA
```

**Code:**

```
MERMAID
Copy
erDiagram
    User ||--o{ State : is located in
```

**Example input / output:**

```

Dry-run for input:

Given Input

User 123, "John Doe", from the state of California

Expected Output in User Table

SQL
Copy
INSERT INTO User (id, username, state_id) VALUES (123, 'John Doe', 5);

where state_id 5 corresponds to the entry for California in the State table.

Expected Insertion in State Table:

SQL
Copy
INSERT INTO State (name, abbreviation) VALUES ('California', 'CA');

If "California" already exists, we would just reference that existing record.
```

**Data:**

```

Sample State table schema:

SQL
Copy
CREATE TABLE State (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    abbreviation CHAR(2)
);

Sample User table schema with a foreign key to the State table:

SQL
Copy
CREATE TABLE User (
    id INT PRIMARY KEY,
    username VARCHAR(50),
    state_id INT,
    FOREIGN KEY (state_id) REFERENCES State(id)
);
```

**Flow:**

1. (User Table)
2. State Table

