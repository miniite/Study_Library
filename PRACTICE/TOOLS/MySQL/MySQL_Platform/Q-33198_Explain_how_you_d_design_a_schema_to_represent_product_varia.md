---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd design a schema to represent product variants (size/color/SKU) with inventory, and keep queries simple for both the catalog page and checkout.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Activity Timeline Schema](<../../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective schema for representing product variants in a database like MySQL, we need to ensure that our data model can handle multiple attributes (size, color) efficiently while also supporting simple queries for both catalog pages and checkouts. Let's break down the problem from first principles.

#### Triage

The core of this question is about designing a schema that balances flexibility with query efficiency. We are asked to represent product variants such as size and color, along with their inventory status. The key aspects include:

Schema Design: How to store variant-specific information.
Query Optimization: Ensuring both catalog pages (read-heavy) and checkouts (write-heavy) can be efficiently served.

#### First-Principles Solution
Step 1: Define the Core Entities

We need three main entities:

Products - Contains basic product details like name, description, category.
Variants - Stores variant-specific attributes such as size and color.
Inventory - Tracks stock availability for each variant.
Step 2: Entity Definitions
Product: A product can have multiple variants but belongs to one category.
Variant: Each variant is uniquely identified by a combination of attribute values (e.g., size, color).
Inventory: Maps each variant to its current stock level and other relevant metrics like reorder points.
Step 3: Normalization

To avoid redundancy, we normalize the schema:

A Product can have many Variants.
Each Variant has a unique combination of attributes.
Variants are linked to their corresponding Products.
Step 4: Data Schema

Here’s how we might represent these entities in MySQL:

SQL
CREATE TABLE Products (
    id INT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category_id INT NOT NULL,
    FOREIGN KEY (category_id) REFERENCES Categories(id)
);

CREATE TABLE Attributes (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE VariantAttributes (
    attribute_id INT,
    value VARCHAR(255),
    UNIQUE KEY (attribute_id, value)
);

CREATE TABLE Variants (
    product_id INT NOT NULL,
    variant_attributes_id INT NOT NULL,
    sku VARCHAR(100) PRIMARY KEY,
    FOREIGN KEY (product_id) REFERENCES Products(id),
    FOREIGN KEY (variant_attributes_id) REFERENCES VariantAttributes(id)
);

CREATE TABLE Inventory (
    variant_sku VARCHAR(100) PRIMARY KEY,
    quantity INT DEFAULT 0,
    reorder_point INT
);

#### Closing Summary
and Common Mistakes/Follow-Ups

By normalizing the schema, we ensure that each attribute combination is stored only once. This makes queries more efficient as they can leverage indexes on Products, Variants, and Inventory. For catalog pages, we can query Products to get an overview, then join with Variants and Inventory for detailed information. During checkouts, we update the Inventory table directly.

#### Common Mistakes
Redundancy: Failing to normalize can lead to duplicated data and inefficient queries.
Indexing: Not properly indexing tables can slow down read operations significantly.
Update Isolation: Ensuring that stock updates are transactional and consistent across related records (e.g., Product, Variant, Inventory).

#### Example
Query for Catalog Page
SQL
SELECT P.id, P.name, V.sku, I.quantity
FROM Products AS P
JOIN Variants AS V ON P.id = V.product_id
JOIN Inventory AS I ON V.sku = I.variant_sku;

This query retrieves products with their variants and current stock levels.

**Comparison:**

| Products                        |
| id | name       | description        | category_id   |
| 1  | T-Shirt    | Black and white.   | 2             |
| Attributes       |
| id | name      |
| 1  | Size      |
| 2  | Color     |
| VariantAttributes            |
| attribute_id | value    |
| 1            | S        |
| 1            | M        |
| 2            | Black    |
| 2            | White    |
| Variants                    |
| product_id | variant_attributes_id | sku         |
| 1          | 3                     | T-S-Black-M |
| 1          | 4                     | T-S-White-M |
| 1          | 5                     | T-S-Black-S |
| Inventory                    |
| variant_sku | quantity | reorder_point |
| T-S-Black-M   | 50       | 20            |
| T-S-White-M   | 30       | 10            |
| T-S-Black-S   | 80       | 30            |

**Code:**

```sql
CREATE TABLE Products (
    id INT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category_id INT NOT NULL,
    FOREIGN KEY (category_id) REFERENCES Categories(id)
);

CREATE TABLE Attributes (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE VariantAttributes (
    attribute_id INT,
    value VARCHAR(255),
    UNIQUE KEY (attribute_id, value)
);

CREATE TABLE Variants (
    product_id INT NOT NULL,
    variant_attributes_id INT NOT NULL,
    sku VARCHAR(100) PRIMARY KEY,
    FOREIGN KEY (product_id) REFERENCES Products(id),
    FOREIGN KEY (variant_attributes_id) REFERENCES VariantAttributes(id)
);

CREATE TABLE Inventory (
    variant_sku VARCHAR(100) PRIMARY KEY,
    quantity INT DEFAULT 0,
    reorder_point INT
);
```

