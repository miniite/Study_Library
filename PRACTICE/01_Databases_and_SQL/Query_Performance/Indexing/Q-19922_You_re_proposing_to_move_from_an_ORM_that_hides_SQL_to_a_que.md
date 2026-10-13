---
type: "practice"
difficulty: "Deep"
---

# You're proposing to move from an ORM that hides SQL to a query-builder with explicit plans reviewed in code review. Make the case to a velocity-focused engineering org, including the migration cost and the failure modes of the ORM-heavy status quo.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are proposing a move from an Object-Relational Mapping (ORM) that abstracts SQL queries to a query-builder with explicit plans reviewed in code review. The goal is to improve database performance and ensure transparency while addressing the velocity concerns of our engineering organization.

#### Triaging the Question
Understanding the Current State: The current ORM-heavy approach is velocity-focused, meaning developers are prioritizing quick development over optimizing queries.
Reviewing the Proposal: We want to switch to a query-builder that exposes SQL directly and mandates code reviews for these plans.

#### Assumptions
The ORM abstraction has led to less optimal database interactions.
The velocity-focused approach has resulted in performance bottlenecks.

#### First-Principles Solution
1. Migration Cost
Initial Setup: Requiring developers to write explicit SQL queries can increase the initial learning curve, which might slow down development in the short term.
Long-term Benefits:
Improved Query Performance: Explicitly optimizing queries can lead to significant performance gains.
Better Code Quality: Forced review of database interactions ensures consistency and reduces bugs.
2. Database Indexing & Query Optimization
Indexing: Ensure that frequently accessed columns are indexed properly. This is crucial for reducing query execution time.
Query Plans: Use query plans to analyze and optimize complex queries. Review these in code reviews to ensure they align with performance goals.
Caching Strategies: Implement caching strategies like query result caching or materialized views to reduce the load on the database.
3. Failure Modes of ORM-Heavy Status Quo
Performance Issues: ORMs often generate suboptimal queries, leading to slower performance and higher resource consumption.
Hidden Bugs: Developers may introduce bugs without realizing the impact on query performance.
Inconsistent Query Behavior: Different developers might write different queries that achieve similar results but have varying performance.

#### Closing Summary

By moving from an ORM-heavy approach to a query-builder with explicit plans, we can ensure better database performance and maintain code quality. While there will be initial costs associated with learning and adapting, the long-term benefits far outweigh these challenges. We must carefully manage this transition to avoid introducing unnecessary complexity or slowing down development.

#### Common Mistakes/Follow-Ups
Ignoring Performance Metrics: Ensuring that we measure query performance before and after migration is crucial.
Over-engineering Queries: Avoid writing overly complex queries just for optimization; balance between simplicity and efficiency.
User Feedback Loop: Continuously gather feedback from developers to refine our approach and address any pain points.

**Comparison:**

```
Phase	Current State	Proposed State
Indexing	Inconsistent, suboptimal	Properly indexed for frequent access
Query Plans	Hidden in ORM	Reviewed and optimized explicitly
Code Quality	Varies by developer	Consistent across the team
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[ORM-Heavy Status Quo] --> B{Performance Issues?}
    B -- Yes --> C[Cached Queries]
    C --> D[Inconsistent Query Behavior]
    A --> E[Limited Review]
    E --> F[Code Quality Concerns]
```

**Example input / output:**

```text
Input:
- Initial database schema with indexing issues.
- Developers using ORM to write queries.

Output:
- Explicit SQL queries written and reviewed in code review.
- Performance metrics showing improved query performance.
- Consistent query behavior across the team.
```

**Data:**

```json
{
  "orm_heavy": {
    "performance_issues": true,
    "hidden_bugs": true,
    "inconsistent_queries": true
  },
  "explicit_sql": {
    "performance_gains": true,
    "better_code_quality": true,
    "optimized_queries": true
  }
}
```

**Flow:**

1. No
2. Initial Setup
3. Write Explicit SQL?
4. Learner Phase
5. Optimize Queries
6. Code Review
7. Performance Metrics
8. Maintain and Refine

