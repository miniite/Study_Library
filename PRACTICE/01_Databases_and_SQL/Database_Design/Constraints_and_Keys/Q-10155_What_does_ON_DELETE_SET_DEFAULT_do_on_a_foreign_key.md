---
type: "practice"
difficulty: "Intermediate"
---

# What does ON DELETE SET DEFAULT do on a foreign key?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Foreign Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Foreign_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ON DELETE SET DEFAULT is an action specified in a foreign key constraint that instructs the database to set the value of the foreign key column(s) to a default value when the referenced primary key is deleted. This mechanism ensures data integrity by maintaining referential consistency, even when the parent record is removed.

#### Triaging the Question

The question asks about ON DELETE SET DEFAULT, which is an action associated with foreign keys in database design. The interviewer might be probing your understanding of how constraints work and how they are used to maintain referential integrity between tables. To answer this question, we'll need to explain what a foreign key constraint is, the different actions available for delete operations on foreign keys, and specifically focus on SET DEFAULT.

#### First-Principles Solution

Foreign Key Basics: A foreign key in SQL is used to establish and enforce links between two tables. It identifies that a column or set of columns can be used to create an association with another table.

Delete Actions on Foreign Keys:

NO ACTION: The deletion will fail if there are any child records referencing the parent record.
CASCADE: All child records referencing the parent record will also be deleted.
SET NULL: The foreign key column(s) in the child table will be set to null when the referenced primary key is deleted.
SET DEFAULT: When a referenced row is deleted, the foreign key column(s) in the child table are automatically set to their default value.

Default Values for Columns:

Default values are specified during the creation of a column using the DEFAULT keyword. For example: column_name DEFAULT 'default_value'.
When ON DELETE SET DEFAULT is applied, if no specific default value is defined for the foreign key columns, the database will use the default value set on those columns.

#### Example Scenario

SQL
CREATE TABLE parent (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE child (
    pid INT,
    FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE SET DEFAULT
);

Behavior When a Parent Record is Deleted:

If the parent table has a row with id = 1 and there are no specific default values defined for the foreign key column in child, all rows referencing pid = 1 will be set to their default value.
For example, if the default value for pid is 0, then any child record previously linked to a parent with id = 1 would now have its pid set to 0.

#### Closing Summary
and Common Mistakes/Follow-Ups
Key Takeaways: ON DELETE SET DEFAULT ensures that the integrity of data in related tables is maintained by setting foreign key columns to their default values when a referenced row is deleted.
Common Mistake: Forgetting to define default values for columns can lead to unexpected behavior, as no value will be set automatically.
Triangulation

The interviewer might ask follow-up questions such as:

How do you determine appropriate default values?
What are the implications of using ON DELETE SET DEFAULT in large databases?
Can this action cause performance issues?

**Code:**

```sql
-- Example SQL to create tables with foreign key constraints and set defaults

CREATE TABLE parent (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

INSERT INTO parent (id, name) VALUES (1, 'Parent A'), (2, 'Parent B');

CREATE TABLE child (
    pid INT,
    FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE SET DEFAULT
);

-- Insert some data into the child table referencing parent records

INSERT INTO child (pid) VALUES (1), (2);
SELECT * FROM child; -- Output: |pid| 1 | 2 |

DELETE FROM parent WHERE id = 1;
SELECT * FROM child; -- Output: |pid| 0 |
```

**Example input / output:**

```text
-- Example input and output

-- Initial state of data in child table before deletion
INSERT INTO child (pid) VALUES (1), (2);
SELECT * FROM child; -- Output: |pid| 1 | 2 |

-- Parent record with id = 1 is deleted, default value for pid is 0
DELETE FROM parent WHERE id = 1;
SELECT * FROM child; -- Output: |pid| 0 |
```

**Data:**

```sql
-- Sample data for tables created above

CREATE TABLE parent (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

INSERT INTO parent (id, name) VALUES (1, 'Parent A'), (2, 'Parent B');

CREATE TABLE child (
    pid INT,
    FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE SET DEFAULT
);

-- Default value for `pid` is 0

DELETE FROM parent WHERE id = 1;
```

