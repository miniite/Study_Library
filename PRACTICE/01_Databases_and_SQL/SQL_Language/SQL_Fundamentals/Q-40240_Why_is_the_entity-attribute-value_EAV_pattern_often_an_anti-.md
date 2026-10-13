---
type: "practice"
difficulty: "Expert"
---

# Why is the entity-attribute-value (EAV) pattern often an anti-pattern, and what are its query costs?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Large Values and TOAST](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The EAV (Entity-Attribute-Value) pattern is often considered an anti-pattern because it can lead to poor performance, complex queries, and difficulties in data integrity and querying. At a high level, EAV splits each piece of information into three separate columns: the entity ID, the attribute name, and the value. While this approach allows for flexibility, it comes with significant drawbacks.

#### Triage

What is being asked: The question seeks to understand why EAV can be problematic from both a performance and data management perspective.
What's being probed: The interviewer wants you to articulate the challenges of querying and managing data in an EAV schema, as well as potential alternatives that might offer better solutions.
Key assumptions: We assume familiarity with relational databases and SQL querying.

#### First-principles solution

Performance Issues:

Joins: Queries in an EAV schema often require multiple joins to retrieve the full set of attributes for a given entity, which can degrade performance significantly as the number of attributes grows.
Indexing: It is challenging to create efficient indexes on partially populated columns (attributes), leading to suboptimal query execution plans.

Complexity:

Query Complexity: Crafting queries that retrieve meaningful data from an EAV schema can be complex and cumbersome, often requiring dynamic SQL generation or stored procedures.
Data Retrieval: Aggregating information or performing calculations across multiple attributes becomes difficult due to the distributed nature of the data.

Data Integrity:

Inconsistent Data Types: Storing all attribute values in a single column allows for inconsistent data types, making it hard to enforce constraints and ensure data consistency.
Validation: Validating and normalizing data is more challenging when attributes are spread across multiple rows.

Maintenance:

Schema Changes: Adding or removing an attribute requires altering the schema, which can be cumbersome in a production environment with existing data.
Data Migration: Migrating data into or out of an EAV schema often involves complex and error-prone processes.

#### Closing summary
+ common mistakes/follow-ups:

#### Common Mistakes

Assuming that EAV is always the best solution for flexible schemas without considering performance implications.
Ignoring normalization benefits in favor of flexibility, leading to degraded query performance.

#### Follow-Ups

Discuss potential alternatives like denormalization or using a document-oriented database (e.g., MongoDB) if true flexibility is needed.

**Comparison:**

```
Entity	Attribute	Value
Product	Price	$59.99
Product	Color	Red
Customer	Price	$49.95
```

**Code:**

```sql
-- Example EAV schema with multiple joins
SELECT e.entity_name, a.attribute_name, v.value
FROM entities e
JOIN entity_attributes ea ON e.id = ea.entity_id
JOIN attribute_values av ON ea.attribute_id = av.attribute_id AND ea.version = av.version;
```

**Example input / output:**

```text
-- Sample input to retrieve the price and color of a product with id = 1
SELECT e.entity_name, av.value AS attribute_value
FROM entities e
JOIN entity_attributes ea ON e.id = ea.entity_id AND e.id = 1
JOIN attribute_values av ON ea.attribute_id = av.attribute_id AND ea.version = av.version
WHERE av.attribute_id IN (101, 102);

-- Expected output
+-----------------+----------------+
| entity_name     | attribute_value |
+-----------------+----------------+
| Product         | 59.99           |
| Product         | Red             |
+-----------------+----------------+
```

**Data:**

```json
{
  "entities": [
    {"id": 1, "entity_name": "Product"},
    {"id": 2, "entity_name": "Customer"}
  ],
  "attributes": [
    {"id": 101, "attribute_name": "Price", "data_type": "numeric"},
    {"id": 102, "attribute_name": "Color", "data_type": "string"}
  ],
  "values": [
    {"entity_id": 1, "attribute_id": 101, "value": 59.99, "version": 1},
    {"entity_id": 1, "attribute_id": 102, "value": "Red", "version": 1},
    {"entity_id": 2, "attribute_id": 101, "value": 49.95, "version": 1}
  ]
}
```

**Flow:**

1. Entities Table
2. Entity Attributes Table
3. Attribute Values Table

