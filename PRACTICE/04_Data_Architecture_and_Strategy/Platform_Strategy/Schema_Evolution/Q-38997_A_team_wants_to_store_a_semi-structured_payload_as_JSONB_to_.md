---
type: "practice"
difficulty: "Deep"
---

# A team wants to store a semi-structured payload as JSONB 'to avoid future migrations'. Argue for and against, and describe when JSONB becomes a schema-evolution liability rather than a convenience.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrations with Read Your Writes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address whether storing semi-structured payloads as JSONB is a good idea in terms of schema evolution and migration strategy, we need to consider several factors, including data consistency, query performance, maintainability, and potential liabilities down the line.

#### Triage

The core question here revolves around evaluating the trade-offs between using JSONB (a column type in PostgreSQL) for storing semi-structured data versus maintaining a more traditional schema with normalized tables. The key points we need to explore include:

Data Consistency and Validation: How do we ensure that the JSON data adheres to expected schemas?
Query Performance: What are the implications of querying large JSONB columns compared to normalized structures?
Maintainability and Scalability: How does maintaining a JSONB schema compare to a well-structured, normalized database over time?

#### First Principles Solution
For Storing Semi-Structured Payloads as JSONB:

Flexibility and Future-Proofing:

Advantage 1: Schema Flexibility: JSONB allows for dynamic updates and additions of fields without the need to alter the underlying database schema, which can be cumbersome with traditional table structures.

#### Example
A user profile in a social network might include various optional fields like interests, hobbies, etc. These fields can evolve over time as new features are added.

Ease of Use:

Advantage 2: Simplified Data Ingestion and Transformation: JSONB makes it easier to ingest semi-structured data from external sources such as APIs or unstructured logs.

#### Example
Real-time ingestion of log data can be quickly stored in a JSONB column without the need for complex ETL processes.

Reduced Schema Evolution Effort:

Advantage 3: Reduced Maintenance Overhead: By deferring schema changes to the application layer, you avoid frequent database schema migrations and potential downtime during upgrades.

#### Example
Adding a new field to an existing JSONB column can be done by updating the application logic without touching the database.
Against Storing Semi-Structured Payloads as JSONB:

Data Consistency and Validation:

Disadvantage 1: Lack of Enforced Constraints: Without proper validation, data stored in JSONB columns might become inconsistent over time.

#### Example
If multiple parts of an application write to the same JSONB column without adhering to a common schema, it can lead to data integrity issues.

Query Performance:

Disadvantage 2: Inefficient Query Patterns: Querying JSONB columns can be less efficient compared to normalized schemas, especially when you need to filter or aggregate specific fields.

#### Example
A query like SELECT * FROM users WHERE hobbies LIKE '%guitar%' on a table with a JSONB column for interests might require a full table scan and an inefficient JSON search.

Maintainability and Scalability:

Disadvantage 3: Increased Maintenance Overhead: As the application evolves, managing and querying complex JSON structures can become cumbersome.

#### Example
Managing updates to nested JSONB fields across multiple tables or views might require a significant amount of application logic.

#### Closing Summary

In conclusion, while JSONB offers flexibility for handling semi-structured data and reducing schema evolution efforts, it comes with its own set of challenges. The key is finding the right balance depending on your specific use case. For applications where dynamic fields are common and data consistency can be enforced through application logic, JSONB can indeed provide a convenient solution. However, for complex queries or scenarios requiring high data integrity and maintainability, traditional normalized schemas may offer better performance and ease of maintenance.

#### Common Mistakes
and Follow-Ups
Mistake 1: Over-reliance on JSONB without proper validation logic.

#### Follow-Up
Implement robust application-level validation to enforce data consistency.
Mistake 2: Ignoring query optimization for large JSONB columns.

#### Follow-Up
Use indexing strategies and partial JSON queries to improve performance.

