---
type: "practice"
difficulty: "Basic"
---

# What is the difference between an entity type and an entity instance?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ER Modeling Fundamentals](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/ER_Modeling_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database context, understanding the difference between an entity type and an entity instance is crucial for designing effective schemas and ER (Entity-Relationship) diagrams. An entity type represents a class or category of things that are of interest to us in the domain being modeled, while an entity instance is a specific occurrence or record of that entity type within our database.

To dive deeper into these concepts:

**Comparison:**

```
Employee_ID	First_Name	Last_Name	Department
101	John	Doe	HR
```

**Code:**

```
MERMAID
Copy
classDiagram
    class Entity_Type {
        +name: string
        +attributes: List<string>
    }
    class Entity_Instance {
        -entityType: Entity_Type
        +id: int
        +values: Map<string, any>
    }
```

**Example input / output:**

```

Let's consider an example to illustrate the difference:

Entity Type: Employee

Attributes could include: employee_id, first_name, last_name, department, etc.

Entity Instance: A specific record in the database that represents an employee

For instance, if we have a table in our database with data for a single employee:

SQL
Copy
INSERT INTO Employees (employee_id, first_name, last_name, department)
VALUES ('101', 'John', 'Doe', 'HR');

Here, Employee is the entity type, and '101', 'John', 'Doe', and 'HR' are attributes of a specific employee instance.
```

