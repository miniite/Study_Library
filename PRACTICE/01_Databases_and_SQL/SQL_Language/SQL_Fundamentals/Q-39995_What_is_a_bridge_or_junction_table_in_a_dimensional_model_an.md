---
type: "practice"
difficulty: "Basic"
---

# What is a bridge or junction table in a dimensional model, and when would a data engineer need one?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validating Join Cardinality](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Validating_Join_Cardinality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a dimensional model used in data warehousing and business intelligence systems, a bridge or junction table acts as an intermediary to establish many-to-many relationships between dimensions. This is crucial because traditional star or snowflake schemas often rely on one-to-many relationships, which may not fully capture complex real-world scenarios.

#### Triaging the Question

The interviewer is probing your understanding of relational database design and how it applies to data warehousing environments. Specifically, they want to see if you can explain when a many-to-many relationship arises and why a bridge table might be necessary. They are also likely interested in your ability to articulate real-world examples where such relationships occur.

#### First-Principles Solution

In dimensional modeling, dimensions represent descriptive attributes of the fact tables (e.g., customers, dates, products). A many-to-many relationship between two dimensions means that a single dimension can be related to multiple records on another dimension. For example, in an e-commerce application, a product could belong to multiple categories (e.g., electronics and home appliances), while a category might contain many different products.

A bridge table is used when there's no natural one-to-many relationship between dimensions. It typically contains foreign keys from both sides of the many-to-many relation. In our example, a ProductCategoryBridge table would have columns referencing product_id and category_id, allowing for multiple products to belong to multiple categories.

Real-World Example

#### Cons
ider an e-commerce platform where customers can buy multiple items in different categories (e.g., clothing and electronics). Without a bridge table, creating such a relationship using traditional star schema would be challenging. You might need to create an intermediate fact table that links customer_id with both product_id and category_id, but this could complicate the query logic.

#### Closing Summary

Bridge tables are essential in dimensional models when dealing with many-to-many relationships between dimensions, ensuring flexibility and accuracy in data representation without complicating the overall schema structure. Understanding their necessity and application is crucial for designing effective data warehousing solutions.

#### Common Mistakes/Follow-Ups
Confusion with One-to-One or Many-to-One Relationships: Sometimes, candidates might mix up one-to-one or many-to-one relationships with many-to-many.
Ignoring Real-World Examples: Not providing concrete examples can make the explanation less convincing. Always relate concepts to practical scenarios for better understanding.

**Comparison:**

```
Products	Categories	ProductCategoryBridge
id	name	product_id
1	Smartphone	1
2	Laptop	1
		2
```

**Code:**

```sql
CREATE TABLE Product (
    product_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE Category (
    category_id INT PRIMARY KEY,
    name VARCHAR(255)
);

-- Bridge table to handle many-to-many relationship
CREATE TABLE ProductCategoryBridge (
    product_id INT,
    category_id INT,
    FOREIGN KEY (product_id) REFERENCES Product(product_id),
    FOREIGN KEY (category_id) REFERENCES Category(category_id)
);
```

**Example input / output:**

```text
INSERT INTO ProductCategoryBridge (product_id, category_id) VALUES
(1, 101), (1, 102),
(2, 101);
```

**Data:**

```json
{
  "products": [
    {"id": 1, "name": "Smartphone"},
    {"id": 2, "name": "Laptop"}
  ],
  "categories": [
    {"id": 101, "name": "Electronics"},
    {"id": 102, "name": "Computing"}
  ]
}
```

**Flow:**

1. Product
2. ProductCategoryBridge
3. Category

