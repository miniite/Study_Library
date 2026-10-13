---
type: "practice"
difficulty: "Basic"
---

# How do you map a one-to-many relationship into relational tables?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Modeling Relationships and Schemas](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database, mapping relationships between entities is crucial to ensure data integrity and efficient queries. One of the most common types of relationships is the one-to-many (1:N) relationship, where one entity can be related to multiple instances of another entity. For example, in an e-commerce application, a Customer can place many Orders. Mapping this relationship into relational tables involves creating two or more tables and establishing a foreign key in the child table that references the primary key of the parent table.

To illustrate this concept, let's delve into the steps required to map such a relationship using ER diagrams and schema design principles. We'll start by understanding what these components are and then move on to the actual implementation.

**Code:**

```
MERMAID
Copy
erDiagram
    CUSTOMER ||--o{ ORDER : placed
```

**Example input / output:**

```text
# Schema Design

**Customer Table**
- CustomerID (Primary Key)
- FirstName
- LastName
- Email
- PasswordHash
- Address
- Phone

**Order Table**
- OrderID (Primary Key)
- CustomerID (Foreign Key referencing Customer.CustomerID)
- OrderDate
- TotalAmount
```

**Data:**

```text
We will define two tables: `Customer` and `Order`. The `Customer` table will have a primary key, typically an auto-incremented ID. The `Order` table will contain a foreign key that references the `CustomerID` in the `Customer` table.
```

