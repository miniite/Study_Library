---
type: "practice"
difficulty: "Deep"
---

# What are the downsides of using a UUID as a primary key compared to an integer?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Primary and Unique Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

UUIDs and integers both have their use cases, but when it comes to primary keys, there are several downsides to using UUIDs over integers. This answer will walk through the core concepts of primary keys and foreign keys, explain why integers are often preferred for primary keys, and explore the specific issues that arise with UUIDs.

**Comparison:**

```
Integer PK	UUID PK
Smaller Storage Footprint	Larger Storage Footprint
Faster Indexing & Sorting	Slower Indexing & Sorting
Easier to Generate Sequentially	Harder to Generate Sequentially
```

**Flow:**

1. Integer PK
2. Faster Indexing
3. Smaller Storage Footprint
4. Easier to Generate Sequentially
5. Faster JOINs in Queries
6. UUID PK
7. Larger Storage Footprint
8. Slower Indexing & Sorting
9. Harder to GenerateSequentially

