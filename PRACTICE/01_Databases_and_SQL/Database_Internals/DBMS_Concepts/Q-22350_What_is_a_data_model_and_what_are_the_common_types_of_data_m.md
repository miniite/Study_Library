---
type: "practice"
difficulty: "Basic"
---

# What is a data model and what are the common types of data models?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Relationship Modeling](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Relationship_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A data model is a way to represent, organize, and manage data in a structured format within a database system. Common types include Entity-Relationship (ER) diagrams, Relational, Network, and Hierarchical models.

In the context of Database Systems, a data model serves as a blueprint for organizing and managing data. It defines how data is stored, accessed, and manipulated. The choice of data model influences the structure and performance of the database system. Let's dive deeper into the common types:

Entity-Relationship (ER) Model: This type of data model focuses on representing entities and their relationships in a conceptual form. Entities represent objects or concepts that are stored, and relationships define how these entities interact with each other.
Relational Model: Widely used today, this model is based on the mathematical concept of relations and uses tables to store and manipulate data. Each table has rows (tuples) and columns (attributes), and a primary key uniquely identifies records.
Network Model: This model allows many-to-many relationships between entities through the use of links or pointers. It provides more flexibility than the hierarchical model but is less common due to complexity in implementation.
Hierarchical Model: In this structure, data is organized as a tree-like structure where each record has one parent and zero or more children. This model simplifies data retrieval but can become cumbersome for complex applications.

Each type of data model comes with its own strengths and weaknesses, making the choice dependent on the specific requirements of an application.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Entity-Relationship Model] --> B[Entities and Relationships]
    A --> C[Terminology like E/R diagram]

    D[Relational Model] --> E[Tables, Rows, Columns]
    D --> F[Primary Keys for Uniqueness]

    G[Network Model] --> H[Many-to-Many Relationships]
    G --> I[Flexibility but complexity]

    J[Hierarchical Model] --> K[Trees with Parents and Children]
    J --> L[Simpler Structure but Limited Flexibility]
```

