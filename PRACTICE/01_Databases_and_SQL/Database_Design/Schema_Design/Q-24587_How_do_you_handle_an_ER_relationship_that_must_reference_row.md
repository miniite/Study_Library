---
type: "practice"
difficulty: "Deep"
---

# How do you handle an ER relationship that must reference rows in two different tables depending on a type flag, without losing declarative referential integrity?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Polymorphic Associations](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Polymorphic_Associations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this challenge, we need to design a database schema where a single relationship can dynamically reference one of two tables based on a type flag, while maintaining referential integrity (RI). The core solution involves using a polymorphic association or a common lookup table to handle the dynamic referencing. This approach ensures that our ER diagram remains clean and declarative RI is preserved.

First, let's triage the question: The interviewer is probing how you can implement a flexible relationship in a relational database while maintaining integrity. We need to ensure that the chosen design allows for easy maintenance, scalability, and efficient querying.

#### Assumptions
There are two tables (TableA and TableB) that should be referenced based on a type flag.
The type flag can take one of two values: 'A' or 'B'.
The relationship is bidirectional, meaning updates in the referencing table need to be cascaded back.
Step-by-Step Solution:

Identify Common Attributes:

Both TableA and TableB might share common attributes that can help us identify the referenced row. Let’s assume these are id, type, and possibly some other attributes like name.

Create a Join Table with Type Discrimination:

Introduce a third table, dynamic_relationships, which will act as a polymorphic association to handle the dynamic referencing.

Define the Schema:

SQL
CREATE TABLE TableA (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE TableB (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE dynamic_relationships (
    id INT PRIMARY KEY AUTO_INCREMENT,
    type CHAR(1),
    ref_id INT,
    other_attr VARCHAR(255), -- Additional attributes if needed
    FOREIGN KEY (type, ref_id) REFERENCES referenced_objects(type, id)
);

Create a Referenced Objects Table:

This table will be used to dynamically reference either TableA or TableB.
SQL
CREATE TABLE referenced_objects (
    type CHAR(1),
    id INT,
    PRIMARY KEY (type, id)
);

INSERT INTO referenced_objects (type, id) VALUES ('A', 1), ('A', 2);
INSERT INTO referenced_objects (type, id) VALUES ('B', 3), ('B', 4);

Insert Data into dynamic_relationships:

SQL
INSERT INTO dynamic_relationships (type, ref_id, other_attr) VALUES ('A', 1, 'AttrValue');

Querying the Relationship:

To query the relationship based on the type flag, we can use a JOIN operation.
SQL
SELECT dr.id, t.name FROM dynamic_relationships dr
JOIN referenced_objects ro ON dr.type = ro.type AND dr.ref_id = ro.id
LEFT JOIN TableA t ON ro.type = 'A' AND ro.id = t.id;

#### Closing Summary

By using a dynamic_relationships table with type discrimination, we maintain referential integrity and allow for flexible referencing. The schema is straightforward yet powerful in handling dynamic relationships without the need for complex joins or custom logic.

#### Common Mistakes/Follow-ups
Forgetting Referential Integrity: Ensure that your foreign keys are properly set up to enforce RI.
Overcomplicating the Schema: Sometimes, overly complex schemas can lead to maintenance issues. Keeping it simple and clear is crucial.
Ignoring Performance Considerations: While this design works well in small to medium-sized datasets, performance might degrade with large tables or frequent updates.

**Code:**

```sql
CREATE TABLE TableA (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE TableB (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE dynamic_relationships (
    id INT PRIMARY KEY AUTO_INCREMENT,
    type CHAR(1),
    ref_id INT,
    other_attr VARCHAR(255), -- Additional attributes if needed
    FOREIGN KEY (type, ref_id) REFERENCES referenced_objects(type, id)
);

CREATE TABLE referenced_objects (
    type CHAR(1),
    id INT,
    PRIMARY KEY (type, id)
);
```

**Example input / output:**

```text
-- Query to fetch relationship data based on type
SELECT dr.id, t.name FROM dynamic_relationships dr
JOIN referenced_objects ro ON dr.type = ro.type AND dr.ref_id = ro.id
LEFT JOIN TableA t ON ro.type = 'A' AND ro.id = t.id;
```

**Data:**

```sql
INSERT INTO referenced_objects (type, id) VALUES ('A', 1), ('A', 2);
INSERT INTO referenced_objects (type, id) VALUES ('B', 3), ('B', 4);

INSERT INTO dynamic_relationships (type, ref_id, other_attr) VALUES ('A', 1, 'AttrValue');
```

**Flow:**

1. TableA
2. dynamic_relationships
3. TableB
4. 2
5. Type 'B'
6. TableA
7. TableB
8. dynamic_query

