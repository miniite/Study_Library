---
type: "practice"
difficulty: "Intermediate"
---

# Design a soft-delete + restore model across an aggregate with child rows; how do you keep queries clean without every reader remembering the deleted filter, and how do foreign-key constraints and unique indexes behave?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Maintaining Denormalized Views](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/Maintaining_Denormalized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When designing a soft-delete feature in a database model that includes child rows, the key challenge is ensuring that deleted records are not visible to readers while maintaining consistency and integrity through foreign-key constraints and unique indexes. The solution involves using a "deleted" flag or timestamp, careful handling of cascading deletes, and filtering out deleted records on query level.

#### Triaging the Question

The question revolves around implementing a soft-delete mechanism for an aggregate model that has child rows. Specifically, it asks how to manage queries to ensure they do not return deleted records while maintaining data consistency and integrity, especially considering foreign-key constraints and unique indexes.

#### Assumptions
The database is relational.
You are using SQL as the query language.
There is an "aggregate" entity with one or more "child" entities (e.g., a Product with multiple Reviews).

#### First-Principles Solution
1. Define the Data Model:

First, let's define the structure of our aggregate and child tables. Assume we have an e-commerce application where products can have reviews.

SQL
-- Aggregate Entity: Product
CREATE TABLE Products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    deleted_at TIMESTAMP NULL -- Soft delete flag; NULL means not deleted.
);

-- Child Entity: Review
CREATE TABLE Reviews (
    id INT PRIMARY KEY,
    product_id INT,
    content TEXT,
    created_at TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES Products(id)
);
2. Implement the Soft-Delete Mechanism:

To soft-delete a product, we update its deleted_at timestamp to the current time.

SQL
UPDATE Products SET deleted_at = CURRENT_TIMESTAMP WHERE id = ?;

For reviews, if they are associated with a deleted product, you might want to cascade the delete or update their status as well. This can be done by using cascading deletes or updating deleted_at.

3. Querying Without Deleted Records:

To ensure that queries do not return deleted records, we need to add appropriate filters.

SQL
-- Fetch all active products and their reviews
SELECT p.*, r.*
FROM Products p
LEFT JOIN Reviews r ON p.id = r.product_id AND r.deleted_at IS NULL;

-- Fetch a single product and its reviews (with pagination)
SELECT p.*, r.*
FROM Products p
JOIN Reviews r ON p.id = r.product_id AND r.deleted_at IS NULL
WHERE p.id = ?
ORDER BY r.created_at DESC
LIMIT 10 OFFSET ?;
4. Handling Foreign-Key Constraints:

When a product is soft-deleted, any associated reviews should ideally be marked as deleted or cascaded to delete the review as well.

SQL
-- Cascading Delete Example (if enabled in DB)
DELETE FROM Reviews WHERE product_id IN (SELECT id FROM Products WHERE deleted_at IS NOT NULL);

-- OR mark them as deleted
UPDATE Reviews SET deleted_at = CURRENT_TIMESTAMP WHERE product_id IN (SELECT id FROM Products WHERE deleted_at IS NOT NULL);
5. Unique Indexes:

Ensure that unique indexes handle deletions appropriately. For instance, if a review's content is supposed to be unique per product:

SQL
-- Ensure uniqueness across products and reviews
CREATE UNIQUE INDEX idx_reviews_unique_content ON Reviews (product_id, content) WHERE deleted_at IS NULL;

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Mistake 1: Failing to update foreign-key constraints properly can lead to orphaned records.
Mistake 2: Not filtering out soft-deleted entities in queries can expose sensitive data.

#### Follow-Up
Discussing performance implications and how pagination or caching might impact the solution.
Triaging Further:

#### Performance Considerations
Evaluate whether soft-deletes are better for large datasets versus hard deletes depending on read vs. write patterns.
Data Migration: How will you handle migrating old data from a hard-delete model to a soft-delete one?

**Comparison:**

```
Column	Type	Constraints
id	INT	PK, AUTO_INCREMENT
name	VARCHAR(255)
deleted_at	TIMESTAMP	Nullable, Default NULL
Column	Type	Constraints
id	INT	PK, AUTO_INCREMENT
product_id	INT	FK (Referencing Products.id), Nullable if soft delete in Reviews
content	TEXT
created_at	TIMESTAMP	Not NULL
```

**Example input / output:**

```text
-- Example input and output for filtering reviews by a specific product ID
SELECT p.*, r.*
FROM Products p
LEFT JOIN Reviews r ON p.id = r.product_id AND r.deleted_at IS NULL
WHERE p.id = 1;
```

**Data:**

```json
{
    "Products": [
        {"id": 1, "name": "Laptop", "deleted_at": null},
        {"id": 2, "name": "Smartphone", "deleted_at": null}
    ],
    "Reviews": [
        {"id": 101, "product_id": 1, "content": "Great product!", "created_at": "2023-10-01"},
        {"id": 102, "product_id": 1, "content": "Excellent performance!", "deleted_at": null},
        {"id": 103, "product_id": 2, "content": "Not recommended.", "created_at": "2023-10-02"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Product Soft-Delete
4. Is the product deleted?
5. Mark as deleted
6. Fetch product and reviews
7. Product Review Fetch
8. Fetch all Reviews for deletedProduct

