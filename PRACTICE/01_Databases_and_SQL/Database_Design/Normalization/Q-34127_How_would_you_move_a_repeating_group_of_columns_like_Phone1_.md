---
type: "practice"
difficulty: "Basic"
---

# How would you move a repeating group of columns like Phone1 Phone2 Phone3 into a properly normalized design?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Redundancy and Anomalies](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Redundancy_and_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To properly normalize a schema that includes repeating groups such as Phone1, Phone2, and Phone3, we need to ensure that each piece of data is stored in only one place, thereby eliminating redundancy. In this case, the goal is to transform the current schema into a normalized form where each phone number is associated with an individual record.

#### Triaging the Question
The question is asking how to redesign a schema with repeating columns (like Phone1, Phone2, and Phone3) to follow normalization principles. The key points here are understanding what denormalization means in this context, identifying the relationships between entities, and designing an optimal structure that minimizes redundancy.

#### First-Principles Solution

Identify the Entity: In this case, we need to identify the entity which has multiple phone numbers. Let's assume it’s a Person or Contact.
Create a Separate Table for Phone Numbers: We will create a new table called PhoneNumbers that stores each phone number uniquely.
Establish a Relationship with the Entity: We’ll use a foreign key to link the PhoneNumbers table back to the original entity (Person or Contact).

Here’s the step-by-step process:

Identify the Entity: Determine what the main entity is that has these phone numbers. For instance, if we’re dealing with people who have multiple contact methods, the entity would be a Person.

Create the PhoneNumbers Table:

Columns: id, person_id, phone_number

#### Cons
traints: Ensure id is an auto-incrementing primary key, and both person_id and phone_number are non-null.

Establish a Relationship with the Entity:

Add a foreign key in the PhoneNumbers table to link back to the Person table.
The person_id column will serve as the reference to the main entity.

Here’s how the schema looks after normalization:

SQL
CREATE TABLE People (
  id INT PRIMARY KEY,
  name VARCHAR(100)
);

CREATE TABLE PhoneNumbers (
  id INT PRIMARY KEY AUTO_INCREMENT,
  person_id INT,
  phone_number VARCHAR(20),
  FOREIGN KEY (person_id) REFERENCES People(id)
);

#### Closing Summary
and Common Mistakes: By normalizing the schema, we ensure that data is stored efficiently without redundancy. A common mistake is forgetting to establish the relationship between the PhoneNumbers table and the main entity (People). Another pitfall is not considering all possible phone numbers a person might have (e.g., mobile, home, office) by adding more columns in the PhoneNumbers table or even splitting them into different tables if needed.

**Code:**

```sql
CREATE TABLE People (
  id INT PRIMARY KEY,
  name VARCHAR(100)
);

CREATE TABLE PhoneNumbers (
  id INT PRIMARY KEY AUTO_INCREMENT,
  person_id INT,
  phone_number VARCHAR(20),
  FOREIGN KEY (person_id) REFERENCES People(id)
);
```

