---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd build a canonical mapping to standardize inconsistent category values (IN/India/in) at scale, and why a JOIN to a curated crosswalk beats a giant CASE that grows unmaintainable.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Validating Join Cardinality](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Validating_Join_Cardinality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a canonical mapping to standardize inconsistent category values like "IN", "India", or "in" at scale, I would leverage a curated crosswalk table rather than using a giant CASE statement. This approach ensures maintainability and scalability by centralizing the transformation logic in a single, well-structured location.

Firstly, let's define what a canonical mapping is: it’s a process where you standardize various representations of data to ensure consistency across your database or dataset. In our scenario, we want to convert all mentions of "India" to its canonical form, which could be an ID like 1023 in a lookup table.

#### Triage
and Assumptions

The question is essentially asking how to handle inconsistent data values through a scalable and maintainable SQL solution. The assumptions here are that the database has some level of normalization, possibly with IDs for categories (like country codes), and we need a robust way to handle these variations without cluttering our queries or making them overly complex.

Step-by-Step Solution

Create a Crosswalk Table We start by creating a crosswalk table that maps all the inconsistent values to their canonical forms.

SQL
CREATE TABLE country_crosswalk (
    old_value TEXT,
    canonical_id INT,
    PRIMARY KEY (old_value)
);

Populate the Crosswalk Table This step involves adding entries for each value we want to standardize. For example:

SQL
INSERT INTO country_crosswalk (old_value, canonical_id) VALUES ('IN', 1023), ('India', 1023), ('in', 1023);

Join with the Crosswalk Table When querying data that contains these inconsistent values, we perform a JOIN operation to replace them with the canonical form.

SQL
SELECT
    original_table.*,
    crosswalk.canonical_id
FROM
    original_table
LEFT JOIN
    country_crosswalk ON lower(original_table.country) = lower(country_crosswalk.old_value);
Full Solution

By using a JOIN to the country_crosswalk table, we can dynamically handle any new inconsistencies without modifying our query. For instance:

SQL
-- Example schema for original data
CREATE TABLE countries (
    id INT PRIMARY KEY,
    name TEXT
);

-- Sample data
INSERT INTO countries (id, name) VALUES (1023, 'India'), (1024, 'United States');

-- Join and standardize the country names
SELECT
    c.id,
    CASE
        WHEN lower(c.name) = 'in' OR lower(c.name) = 'india' THEN 1023
        ELSE c.id
    END AS canonical_id
FROM
    countries c;
Advantages of Using a Crosswalk Table
Maintainability: Centralizing the mapping logic in one place makes it easier to maintain and update.
Scalability: Adding new mappings is straightforward, unlike expanding a CASE statement which can become unwieldy.
Readability: Queries remain clean and easy to understand.

#### Closing Summary

Using a curated crosswalk table via JOIN operations over a giant CASE statement not only keeps our queries simpler but also ensures that we can easily manage changes in the future. This approach is particularly useful in large-scale data processing where inconsistencies are common, ensuring high-quality and consistent data throughout the system.

#### Common Mistakes/Follow-Ups
Not Using Indexes: Ensure to index the country_crosswalk table on the old_value column for performance.
Ignoring Case Sensitivity: Always use proper handling of case sensitivity as shown in the examples.
Failure to Update Crosswalk Table: Regularly updating or adding mappings ensures data quality over time.

**Code:**

```sql
CREATE TABLE country_crosswalk (
    old_value TEXT,
    canonical_id INT,
    PRIMARY KEY (old_value)
);

INSERT INTO country_crosswalk (old_value, canonical_id) VALUES ('IN', 1023), ('India', 1023), ('in', 1023);

SELECT
    original_table.*,
    crosswalk.canonical_id
FROM
    original_table
LEFT JOIN
    country_crosswalk ON lower(original_table.country) = lower(country_crosswalk.old_value);
```

**Example input / output:**

```text
-- Sample input data
+----+--------+
| id |  name  |
+----+--------+
| 1023 | India  |
| 1024 | USA    |
+----+--------+

-- Expected output after standardization
+----+-------+------------+
| id | name  | canonical_id|
+----+-------+------------+
| 1023 | India  | 1023       |
| 1024 | USA    |            |
+----+-------+------------+
```

**Data:**

```sql
CREATE TABLE countries (
    id INT PRIMARY KEY,
    name TEXT
);

INSERT INTO countries (id, name) VALUES (1023, 'India'), (1024, 'United States');
```

