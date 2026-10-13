---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd structure keys so that a single Query can fetch a parent entity and all its children in one round-trip (the 'pre-join' pattern) in DynamoDB. What are the limits of this pattern?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DynamoDB Modeling and Capacity](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve fetching a parent entity along with all its children in a single query using DynamoDB, we need to carefully design our key schema to leverage the "pre-join" pattern effectively. This involves structuring our data such that it allows us to fetch related items without having to perform multiple read operations. Here’s how you can do this:

Firstly, let's understand what the pre-join pattern means: It involves storing and retrieving hierarchical or relational data in a single query, which optimizes performance by reducing the number of round-trips required.

Now, I’ll walk through the key steps to structure keys for this purpose.

**Comparison:**

```
Attribute	Type	Description
parentId	String	Unique identifier for the parent entity.
childrenIds	List<String>	A list of unique identifiers for all children entities belonging to this parent.
```

**Code:**

```python
# Example DynamoDB Table Structure

table_name = "ParentChildEntities"

# Define an item as a parent entity with its children IDs
parent_item = {
    "parentId": {"S": "parent1"},
    "childrenIds": {"L": [
        {"S": "child1_id"},
        {"S": "child2_id"}
    ]}
}
```

**Example input / output:**

```

Given a parent ID, say parent1, we would query the table to retrieve both the parent and its children.

Input:

Query with parentId as key

Output:

JSON
Copy
{
  "parentId": "parent1",
  "childrenIds": ["child1_id", "child2_id"]
}

The above JSON represents a single item in DynamoDB that contains the parent ID and an array of child IDs.
```

**Data:**

```json
{
  "parentId": "parent1",
  "childrenIds": ["child1_id", "child2_id"]
}
```

**Flow:**

1. Yes
2. No
3. Query with Parent ID
4. Parent ID exists?
5. Fetch parent and childrenusing single query
6. Error handling: parent doesnot exist
7. Return parent entity withchildren IDs

