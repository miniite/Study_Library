---
type: "practice"
difficulty: "Basic"
---

# How do repeating groups violate First Normal Form?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ First Normal Form](<../../../../01_Databases_and_SQL/Database_Design/Normalization/First_Normal_Form.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how repeating groups violate First Normal Form (1NF), we first need to clarify what 1NF is and why it's important. In a relational database, 1NF ensures that each column contains atomic values—meaning they are indivisible, like integers or strings—and that all rows in the table contain an equal number of columns. Violating 1NF means having data that doesn't meet these criteria.

The question specifically asks about how repeating groups violate 1NF. Let's break this down step by step:

#### Triage
What is being asked: The core of the question is to explain why repeating group structures in a database table violate 1NF.
What the interviewer is probing: They are interested in your understanding of atomic values and how they apply to each row in a table. They also want to ensure you understand the importance of uniformity across rows for data integrity.
Reasoning from First Principles

Understanding Atomic Values:

In 1NF, every column must contain a single value that cannot be further divided.
For example, if we have a Person table with columns like Name, Address, and PhoneNumbers, each of these fields should hold only one piece of information.

Repeating Groups:

Repeating groups are structures in which multiple values are stored in a single column.
For instance, if you have a Person table with a Hobbies column that stores a list of hobbies for each person, this is an example of repeating data within a column.

Why It Violates 1NF:

In the case of the Hobbies column, it's storing multiple values (hobbies) in one cell.
This means that the value in the Hobbies column for each row is not atomic but rather a list or string containing multiple pieces of data separated by commas, spaces, or some other delimiter.

#### Example

#### Cons
ider a simple table structure:

SQL
CREATE TABLE Person (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Hobbies VARCHAR(255)
);

If the Hobbies column contains multiple values like "Reading, Traveling, Swimming", this violates 1NF because it is not storing a single atomic value. Instead, it should have separate records for each hobby.

#### Solution

To fix this and bring the table into compliance with 1NF, you would denormalize the data by creating a new Hobby table linked to the Person table:

SQL
CREATE TABLE Person (
    ID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Hobby (
    ID INT PRIMARY KEY,
    Description VARCHAR(50),
    PersonID INT,
    FOREIGN KEY (PersonID) REFERENCES Person(ID)
);

Now, each hobby is stored as a separate record with a foreign key linking it back to the Person table. This ensures that each column contains only one piece of data.

#### Closing Summary

Repeating groups in a database violate 1NF because they store multiple values within a single cell, which breaks the requirement for atomicity. To resolve this issue, you should denormalize the data by creating additional tables to ensure each row has uniform and indivisible values.

#### Common Mistakes/Follow-Ups
Not Understanding Atomicity: A common mistake is misunderstanding what constitutes an "atomic" value.
Ignoring Other Normal Forms: After addressing 1NF, interviewers might probe further into higher normal forms (2NF, 3NF) to understand the full scope of data integrity.

**Comparison:**

```
ID	Name
1	Alice
2	Bob
ID	Description	PersonID
1	Reading	1
2	Traveling	1
3	Swimming	1
4	Cooking	2
5	Painting	2
6	Hiking	2
```

**Code:**

```sql
CREATE TABLE Person (
    ID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Hobby (
    ID INT PRIMARY KEY,
    Description VARCHAR(50),
    PersonID INT,
    FOREIGN KEY (PersonID) REFERENCES Person(ID)
);
```

**Example input / output:**

```text
Input:
- Table: Person with a column 'Hobbies' containing multiple values

Output:
- New table structure:
  - Person (ID, Name)
  - Hobby (ID, Description, PersonID)

Explanation of changes:
1. The original Hobbies column is split into two tables.
2. A foreign key in the Hobby table links it to the Person table.
```

**Data:**

```json
{
  "persons": [
    { "id": 1, "name": "Alice", "hobbies": "Reading, Traveling, Swimming" },
    { "id": 2, "name": "Bob", "hobbies": "Cooking, Painting, Hiking" }
  ]
}
```

**Flow:**

1. Person Table
2. Hobbies Column
3. Hobbies as a List
4. Duplicate Values
5. Hobbies Normalized
6. New Hobby Table withPersonID

