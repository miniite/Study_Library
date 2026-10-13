---
type: "practice"
difficulty: "Deep"
---

# How do you handle a schema-less write where different item types share a table in DynamoDB (single-table design)? How do you keep application code sane and avoid attribute-name collisions?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DynamoDB Modeling and Capacity](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a wide-column NoSQL database like DynamoDB, using single-table design can be powerful but requires careful handling to avoid schema conflicts and maintain clean, scalable applications. The key challenge is managing different item types within the same table without attribute name collisions, which could lead to confusion or errors in application logic.

The question is really asking about how to design a database schema that allows for flexibility while ensuring data integrity and code maintainability. We need to consider the following:

Schema Flexibility vs. Data Integrity: How to balance the dynamic nature of NoSQL with consistent, predictable access patterns.
Attribute Name Collisions: Ensuring unique attribute names across different item types.
Application Code Maintainability: Designing application logic that can easily handle various data types without introducing bugs or complexity.

To address these challenges, we'll start from first principles and build a solution step by step. We will leverage naming conventions, consistent attribute structures, and partitioning strategies to maintain clarity and prevent collisions.

Firstly, let's outline the key steps:

Use Naming Conventions: Establish clear naming patterns for attributes specific to each item type.
Define Common Attributes: Identify shared fields that can be used across multiple types.
Partition by Type: Use a consistent approach to partition items of different types within the same table.

#### Cons
istent Data Model: Ensure a uniform data model is applied where possible, reducing complexity.

Let's delve into these steps in more detail:

Step 1: Naming Conventions

Using meaningful and consistent naming conventions can help prevent attribute name collisions. For instance, you might prefix attributes with the item type to ensure uniqueness:

user_id
customer_name
product_type (e.g., book_title, movie_director)

This approach makes it clear which part of an object is being referenced and helps in writing more readable code.

Step 2: Define Common Attributes

Identify attributes that are common across multiple item types. For example, in a system where you have both users and products:

Common Fields: id, createdAt, updatedAt
User-Specific Fields: username, email, passwordHash
Product-Specific Fields: title, price, description

By clearly defining common fields, we can ensure that these attributes are used consistently across different item types.

Step 3: Partition by Type

Use partition keys or sort keys to group items of the same type together. For example:

If you have a table called items with an attribute type, you could use it as the partition key.

#### Example
partition key: type#user|id

#### Example
item entry: { "partitionKey": "type#user|123", "username": "john_doe", "email": "john@example.com" }

This approach ensures that items of the same type are logically grouped, making it easier to query and manage them.

Step 4: Consistent Data Model

Adopt a consistent data model where possible. For example, if you frequently need to retrieve user details alongside related products or vice versa, consider structuring your data in a way that facilitates such operations:

Use nested structures for related objects (e.g., embed user attributes within product objects).
Define a base schema with common fields and extend it as needed.

#### Closing Summary

By leveraging naming conventions, defining common fields, partitioning by type, and maintaining consistent data models, we can effectively handle different item types in DynamoDB without running into attribute name collisions. This approach ensures that the application code remains sane and maintainable, while still allowing for flexible and dynamic data storage.

#### Common mistakes
include not establishing clear naming patterns, failing to define common fields, neglecting partitioning strategies, or adopting inconsistent data models that increase complexity and introduce bugs. Always validate assumptions through real-world testing scenarios and consider edge cases such as concurrent updates or complex queries.

**Comparison:**

| Item Type | Common Fields        | Example Attributes                       |
|-----------|----------------------|-----------------------------------------|
| User      | id, createdAt, updatedAt | username, email                         |
| Product   | id, createdAt, updatedAt | title, price, description                |

**Code:**

```python
# Example of a Python class for DynamoDB Item
class User:
    def __init__(self, user_id, username, email):
        self.partition_key = f"type#user|{user_id}"
        self.username = username
        self.email = email

class Product:
    def __init__(self, product_id, title, price, description):
        self.partition_key = f"type#product|{product_id}"
        self.title = title
        self.price = price
        self.description = description

user = User(123, "john_doe", "john@example.com")
product = Product(456, "Python Programming Book", 39.99, "A comprehensive guide to Python.")
```

**Example input / output:**

```text
Input:
- User creation request: { "partitionKey": "type#user|123", "username": "john_doe", "email": "john@example.com" }

Output:
- Inserted user item into DynamoDB with partition key 'type#user|123' and attributes username='john_doe', email='john@example.com'
```

**Data:**

```

{ "partitionKey": "type#user|123", "username": "john_doe", "email": "john@example.com" }

CODE
Copy

```json
{
    "partitionKey": "type#product|456",
    "title": "Python Programming Book",
    "price": 39.99,
    "description": "A comprehensive guide to Python."
}
```

**Flow:**

1. Yes
2. No
3. Receive Write Request
4. Is Item Type Known?
5. Generate Partition Key
6. Write to DynamoDB
7. Reject or Handle UnknownType
8. Return Success Response

