---
type: "practice"
difficulty: "Expert"
---

# Explain how UUID v7 or ULID improves index locality compared to UUID v4, and when you'd still avoid it as a clustered key.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Primary and Unique Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When choosing between UUIDs (version 4, version 7, or ULID) for indexing in databases, one crucial aspect is index locality, which impacts read performance and query efficiency. In this context, we need to understand how the structure of these IDs affects their distribution within the table.

UUID v4 generates 128-bit random numbers, which can lead to poor spatial distribution, making it less ideal for clustered indexes in many cases. On the other hand, UUIDs version 7 and ULID provide more structured time-based values that are generally better suited for indexing on a temporal or sequential basis.

TL;DR: UUID v7 and ULID improve index locality by leveraging their structured nature, whereas UUID v4 often results in random distribution, which is suboptimal for clustered indexes.

The question asks us to compare the indexing benefits of UUID v7 (and its variant ULID) with UUID v4. Specifically, we need to explain how these ID formats improve index locality and when we might avoid using them as a clustered key. The core idea is to leverage their structured nature for better spatial distribution.

UUID v4: Generates 128-bit random values, which do not provide any inherent order or predictability.
UUID v7: Combines randomness and time-based components. It uses the most significant bits (MSB) for time, ensuring a certain degree of sequence in older UUIDs.
ULID (Universally Unique Lexicographically Sortable Identifier): Similar to UUID v7 but uses base-62 encoding with time as the MSB, providing an even more structured and lexicographically sortable value.
Index Locality

Index locality refers to how well an index is distributed spatially within a table. For read-heavy queries, especially range-based or sequential scans, good index locality can significantly improve performance by reducing the number of disk I/O operations required.

UUID v4
Random Distribution: Each UUID v4 generated value is random and does not follow any predictable order.
Poor Index Locality: This randomness means that rows are scattered across different blocks in storage, leading to more costly page and row lookups during queries.
UUID v7 & ULID
Structured Time Component: The MSB of these IDs contains a timestamp, which provides a natural ordering.
Better Index Locality: By having an inherent order based on the timestamp, rows are more likely to be physically close to each other in storage, leading to better spatial distribution and improved index performance.
When to Avoid UUID v7/ULID as Clustered Keys

Even though UUID v7 and ULID offer better index locality compared to UUID v4, there are scenarios where you might still prefer a different strategy:

High Update Rates: If the table experiences frequent updates or deletes, using a time-based ID can lead to hotspots. The clustered index needs to be rebuilt often, which can cause significant performance overhead.
Complex Queries: In cases where queries involve complex join operations or filtering on non-time-related fields, relying solely on UUID v7/ULID as the clustering key may not be optimal.
Legacy Systems: If your existing system heavily relies on specific IDs for business logic or third-party integrations, changing to a time-based ID might introduce significant refactoring efforts.

#### Closing Summary
and Common Mistakes

In summary, while UUID v7 and ULID offer improved index locality by providing a structured timestamp component, it's important to weigh this benefit against other factors such as update frequency and query complexity. Always consider the overall performance needs of your application before making a choice.

#### Common mistakes
include overestimating the benefits without considering potential downsides or underestimating the effort required for migration if you decide to change from a random UUID v4.

**Code:**

```sql
-- Example SQL for creating an index using UUID v7 (or ULID)
CREATE INDEX idx_example ON example_table(uuid_v7_column);
```

