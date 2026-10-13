---
type: "practice"
difficulty: "Deep"
---

# How do you decide, as an architect, between a partial covering index and a full one for a soft-delete-heavy table, and how do you keep the choice consistent across the estate?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Covering Indexes and Index Strategy for Sharding](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/Covering_Indexes_and_Index_Strategy_for_Sharding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As an Architect focusing on database indexing strategies for a soft-delete-heavy table, the decision between partial covering indexes and full ones hinges on several critical factors including performance optimization, query patterns, maintenance overhead, and consistency across the estate. The goal is to balance these aspects while ensuring that decisions are informed by data-driven insights.

#### Triage

The question is essentially asking how to design an indexing strategy for a table that frequently performs soft deletions (i.e., marking records as deleted rather than physically removing them). We need to consider whether partial covering indexes, which include columns used in queries but exclude non-essential ones, are more appropriate compared to full covering indexes. Additionally, ensuring consistency in the choice of indexes across similar tables and databases is crucial for maintaining a cohesive design.

#### Assumptions
The table has frequent soft deletion operations.
Query patterns vary significantly among different parts of the application.
Indexes have an impact on write performance due to their maintenance overhead.

#### Cons
istency in indexing decisions across the estate is important for maintainability and scalability.

#### First-Principles Solution

To make an informed decision, we must first understand the nature of queries executed against this table and how indexes can optimize them. We will categorize queries into those that need a partial covering index and those that benefit from a full one.

Step 1: Analyze Query Patterns

Identify common query patterns:

Queries frequently using WHERE clauses for soft delete status.
Selective queries fetching only the necessary columns (partial covering).
Aggregations and joins requiring all columns (full covering).
Step 2: Evaluate Partial Covering Indexes

Partial covering indexes are ideal when a query needs to retrieve multiple columns, but not all. For example:

SQL
CREATE INDEX idx_partial ON soft_delete_table (id, deleted_at) INCLUDE (created_at);

Here, deleted_at is included because it's often used in the WHERE clause for filtering, while created_at might be necessary for some queries.

Step 3: Consider Full Covering Indexes

Full covering indexes are beneficial when a query needs all columns but can use a single index to satisfy the request. For instance:

SQL
CREATE INDEX idx_full ON soft_delete_table (id, deleted_at, created_at);

This index covers id, deleted_at, and created_at, which might be needed for various queries.

Step 4: Balance Performance and Write Overhead

Partial covering indexes reduce the size of the index, which can improve write performance. However, they may require more index scans to retrieve all required columns. Full covering indexes are more efficient in read scenarios but incur higher maintenance costs during updates.

#### Closing Summary

The choice between partial and full covering indexes depends on query patterns and the balance between read and write performance. A well-rounded approach involves:

Categorizing queries based on their needs.
Selecting appropriate index types to optimize for both scenarios.
Ensuring consistency across similar tables by following a standardized naming convention and documentation.

#### Common Mistakes/Follow-Ups
Over-indexing: Creating too many indexes can degrade write performance significantly.
Ignoring read performance: Focusing solely on write operations without considering query optimization.

#### Cons
istency issues: Failing to maintain uniform indexing strategies across the estate, leading to redundant or conflicting index definitions.

#### Triage
Recap

To decide between partial covering and full indexes:

Analyze query patterns.
Choose appropriate index types based on read/write needs.
Ensure consistency in design and documentation.

