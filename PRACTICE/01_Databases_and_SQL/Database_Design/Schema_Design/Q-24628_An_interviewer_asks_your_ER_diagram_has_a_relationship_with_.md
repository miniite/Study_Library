---
type: "practice"
difficulty: "Intermediate"
---

# An interviewer asks: 'your ER diagram has a relationship with no attributes and 1:1 both ways — is that a smell?' — how do you respond and when is it legitimate?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ ER Modeling Fundamentals](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/ER_Modeling_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database design, an entity-relationship (ER) diagram without any attributes on a relationship can indeed be seen as a potential "smell," indicating that something might need further clarification or refinement. However, this is not always a bad sign; it's legitimate under certain conditions.

The core of the question revolves around understanding what such a 1:1 relationship means and whether it should exist without attributes. A 1:1 relationship implies that each entity in one set is associated with exactly one entity in another set, and vice versa. The key here is to ensure the relationship has meaning beyond just existence.

To properly address this, let’s break down the solution from first principles:

#### Triaging the Question

The interviewer is probing your understanding of ER diagram design practices, specifically focusing on 1:1 relationships without attributes. They are likely concerned that such a structure might indicate poor database normalization or an overly complex model.

#### Assumptions
and Context
The relationship between two entities (say EntityA and EntityB) has been modeled as 1:1.
There are no specific attributes defined on this relationship in the ER diagram.
Step-by-Step Solution
1. Purpose of the Relationship
First, determine why a 1:1 relationship is necessary. For example, it might represent an identity relationship where EntityA and EntityB share some intrinsic connection (e.g., a person’s name in two different tables).
2. Entity and Attribute Analysis
Check the attributes of both entities to see if there are any shared or dependent attributes that could be moved into a junction table, thereby eliminating the need for an attributeless relationship.

#### Cons
ider whether this relationship is essential at all. Sometimes, 1:1 relationships might be overkill; perhaps one entity can simply inherit attributes from another.
3. Normalization and Semantics
Ensure that the database remains in a properly normalized state. If the absence of attributes leads to data redundancy or anomalies (like insertions or deletions causing orphaned records), consider revising the design.
Evaluate if the relationship is purely semantic, meaning it exists only for conceptual clarity without any functional impact.
4. Real-World Example
Suppose we have two entities: Student and Class. A student can be in exactly one class at a time, and vice versa. The relationship could be defined as 1:1 but not require an attribute since it’s just about the existence of a pairing.

#### Conclusion

A 1:1 relationship without attributes is legitimate when:

It is purely for conceptual clarity.
There are no shared or dependent attributes that need to be represented on the relationship itself.
The relationship does not violate normalization rules and does not introduce redundancy.

However, if such relationships start cluttering your model unnecessarily, it might indicate an overly complex design. In practice, you should strive for a balance between clear semantics and efficient data management.

#### Common Mistakes
and Follow-Ups
Mistake 1: Overlooking the potential for shared attributes that could be moved to a junction table.
Mistake 2: Assuming all 1:1 relationships must have attributes, which can lead to unnecessary complexity.

#### Follow-Up
The interviewer might ask about specific normalization rules or whether such designs impact query performance. Be prepared to discuss both the theoretical and practical implications.

**Code:**

```
MERMAID
Copy
erDiagram
    Student ||--o{ Class : takes part in
```

