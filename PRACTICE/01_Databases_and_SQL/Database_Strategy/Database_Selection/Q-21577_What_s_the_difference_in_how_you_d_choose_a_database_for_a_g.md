---
type: "practice"
difficulty: "Deep"
---

# What's the difference in how you'd choose a database for a green-field product versus a re-platforming of a legacy monolith, and how does existing team skill and inertia weight the second case?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choosing a Database for the Workload](<../../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When selecting a database system for a new project (green-field) or re-platforming an existing one, the primary factors include the nature of the data, query patterns, transactional requirements, scalability needs, and team expertise. However, in a green-field scenario, there's more flexibility to choose a database that aligns perfectly with the application’s requirements. In contrast, during re-platforming, existing skills and organizational inertia often constrain the choice, necessitating a trade-off between modernization benefits and operational continuity.

**Comparison:**

```
Factor	Green-field	Legacy re-platform
Data model	Flexible schema	Constrained by existing codebase
Query patterns	Complex aggregations, real-time streaming	Simple select statements, batch processing
Team skill and inertia	Minimal impact	Significant impact
```

**Example input / output:**

```text
Input (Green-field):
- Data Model: Flexible schema, high-tech stack
- Query Patterns: Complex aggregations, real-time streaming

Output (Recommended Database):
- NewSQL or NoSQL for flexibility and performance
- Example: Cassandra or CockroachDB for large-scale, distributed data

Input (Legacy re-platforming):
- Data Model: Constrained by existing codebase
- Query Patterns: Simple select statements, batch processing

Output (Recommended Database):
- Relational database with transactional support
- Example: PostgreSQL or Oracle due to legacy skill set and familiar API
```

**Data:**

```json
{
  "data_model": {
    "green_field": ["flexible_data_schema", "high_tech_stack"],
    "legacy_replatform": ["constrained_by_existing_codebase", "limited_skill_set"]
  },
  "query_patterns": {
    "green_field": ["complex_aggregations", "real-time_streaming"],
    "legacy_replatform": ["simple_select_statements", "batch_processing"]
  }
}
```

**Flow:**

1. Flexibility & Performance
2. Existing Skills & Familiarity
3. Green-field Selection
4. NewSQL/NoSQL/New
5. Cassandra/CockroachDB
6. Legacy re-platformingSelection
7. RDBMS with transactionalsupport
8. PostgreSQL/Oracle

