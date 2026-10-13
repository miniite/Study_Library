---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a natural key and a surrogate key at a beginner level?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Surrogate Keys in Normalization](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between a natural key and a surrogate key, let's start with their definitions and then explore why they are used in database design.

TL;DR:

A natural key is a field or combination of fields that uniquely identifies a record in a table. A surrogate key, on the other hand, is an artificial key (usually an auto-incrementing integer) created for the sole purpose of uniquely identifying records within a table. The main difference lies in their origin and usage.

#### Triage

The question asks about natural keys and surrogate keys at a beginner level. We need to explain these terms in simple terms and highlight the primary differences between them, including their origins and uses.

#### First-Principles Solution
Natural Key

A natural key is derived from real-world attributes of the data that inherently serve as unique identifiers for records. For example, in a customer table, a natural key could be a combination of customer_name and date_of_birth. If no two customers share the same name and date of birth, this pair acts as a natural key.

Advantages:

Directly related to business logic.
Can provide meaningful insights into the data.

Disadvantages:

Not always unique; e.g., if "John Doe" is a common name.
May change over time (e.g., legal changes, updates).
Surrogate Key

A surrogate key is an artificial identifier created by the database designer to uniquely identify records. It has no inherent meaning and does not relate directly to any business logic. Common examples include id, customer_id, or auto-incrementing integers.

Advantages:

Unique, stable, and immutable.
Can simplify queries and joins since they are always unique.

Disadvantages:

Do not provide meaningful insights into the data.
May require additional storage space (e.g., an extra column).

#### Example Scenario

#### Cons
ider a Customer table where we need to uniquely identify customers. Let's look at both approaches:

Natural Key Approach
SQL
CREATE TABLE Customer (
    customer_name VARCHAR(100),
    date_of_birth DATE,
    address VARCHAR(255),
    -- other columns
);

Here, a natural key could be (customer_name, date_of_birth).

Surrogate Key Approach
SQL
CREATE TABLE Customer (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100),
    date_of_birth DATE,
    address VARCHAR(255),
    -- other columns
);

Here, id is a surrogate key used to uniquely identify customers.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, natural keys are based on business attributes and can provide meaningful insights, but they may not always be unique or stable. Surrogate keys offer a reliable way to uniquely identify records without any inherent meaning, making them easier to manage in complex database designs. When designing tables, it's crucial to consider the trade-offs between using natural keys versus surrogate keys based on your specific requirements and constraints.

#### Common mistakes
include:

Using natural keys where they are not truly unique or stable.
Ignoring the need for a unique identifier when natural keys are insufficient.

#### Follow-ups
might involve discussing scenarios where one would prefer natural keys over surrogate keys, such as in cases where meaningful business logic is critical to the design.

**Code:**

```sql
-- Example of creating a table with natural keys
CREATE TABLE Customer (
    customer_name VARCHAR(100),
    date_of_birth DATE,
    address VARCHAR(255)
);

-- Example of creating a table with surrogate keys
CREATE TABLE Customer (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100),
    date_of_birth DATE,
    address VARCHAR(255)
);
```

