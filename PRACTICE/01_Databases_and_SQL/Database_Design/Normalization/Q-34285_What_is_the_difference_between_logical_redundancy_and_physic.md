---
type: "practice"
difficulty: "Deep"
---

# What is the difference between logical redundancy and physical redundancy in a schema?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Transitive Dependencies and 3NF](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Transitive_Dependencies_and_3NF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Logical redundancy and physical redundancy are two types of redundancy in database design, each serving distinct purposes but often leading to different trade-offs.

TL;DR: Logical redundancy involves repeating data across tables within the same database, whereas physical redundancy is about duplicating entire databases or data files for backup and high availability. The primary concern with logical redundancy is data inconsistency, while physical redundancy focuses on maintaining data availability during failures.

Logical Redundancy:

Definition: Repeating the same data in multiple places within a single schema.

#### Example
A CustomerOrders table storing customer ID, product details, and order date, which could be redundant if this information is already stored in the Products or Customers tables.

Physical Redundancy:

Definition: Duplicating entire databases or data files to ensure data availability and prevent loss.

#### Example
Creating a secondary copy of a database on another server for backup purposes, ensuring that even if one instance fails, the other can take over seamlessly.

#### Triage
What is Being Probed

The interviewer likely wants you to distinguish between these two forms of redundancy, understand their implications on data integrity and performance, and discuss when each might be appropriate. They may also probe your understanding of normalization levels and how they relate to both types of redundancy.

#### First-Principles Solution
Logical Redundancy
Definition: Occurs at the database schema level.

#### Example
A CustomerOrders table storing customer ID, product details, and order date, which could be redundant if this information is already stored in the Products or Customers tables.
Implications:
Data Inconsistency: Duplicates can lead to inconsistencies if not managed properly.
Performance Overhead: Increased storage requirements and potential impact on query performance due to additional data copies.
Physical Redundancy
Definition: Occurs at the hardware or network level, involving multiple physical instances of a database.

#### Example
Creating a secondary copy of a database on another server for backup purposes.
Implications:
Data Availability: Ensures data availability in case of server failure.
Maintenance Overhead: Requires additional resources and complexity to manage multiple instances.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, logical redundancy is about avoiding duplicating data within a single schema, which can lead to inconsistencies if not managed properly. Physical redundancy involves replicating databases or data files across different servers for backup and high availability purposes. Both have their trade-offs: logical redundancy focuses on maintaining data integrity, while physical redundancy aims at ensuring availability.

#### Common Mistakes/Follow-Ups

Assuming that all duplication is bad (logical redundancy) without considering the context of consistency vs. performance.
Overlooking the need for both types of redundancy in different scenarios—e.g., using logical normalization to avoid inconsistencies and physical replication for disaster recovery.
Failing to understand the importance of proper database management techniques, such as triggers or constraints, to enforce data integrity in cases of logical redundancy.

#### Example Scenario

Suppose you have a Customers table with customer details and a separate CustomerOrders table storing order information. If the CustomerOrders table were to store additional fields like product ID and price (already present in the Products table), this would be an example of logical redundancy. On the other hand, if you set up a secondary database replica for disaster recovery purposes, that would be physical redundancy.

**Comparison:**

| Customers | CustomerOrders |
|-----------|----------------|
| customer_id | order_id      | product_id  | order_date  |
| name       | quantity      | price       |             |
| address    | customer_id   |            |             |

**Code:**

```sql
-- Example of logical redundancy
CREATE TABLE CustomerOrders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES Customers(customer_id),
    product_id INT REFERENCES Products(product_id),
    order_date DATE,
    quantity INT
);

-- Example of physical redundancy
BACKUP DATABASE mydatabase TO DISK = 'C:\backups\mydatabase_backup.bak';
```

**Example input / output:**

```text
Input (Logical Redundancy):
- Insert customer details into the Customers table.
- Insert order details with product ID and price in the CustomerOrders table.

Output (Potential Inconsistency):
- Product ID and price are duplicated, leading to potential inconsistencies if not managed properly.

Input (Physical Redundancy):
- Backup the main database using a backup tool.

Output (Data Availability):
- A secondary copy of the database is created, ensuring data availability in case of failure.
```

**Data:**

```json
{
  "Customers": {
    "customer_id": 1,
    "name": "John Doe",
    "address": "123 Main St"
  },
  "Products": {
    "product_id": 456,
    "name": "Laptop",
    "price": 899.99
  }
}
```

**Flow:**

1. Logical Redundancy
2. (Avoid data duplication withina schema)
3. (Data inconsistency andperformance overhead)
4. Ensure proper managementthrough constraints
5. (Maintain data integrity)
6. Physical Redundancy
7. (Replicate databases forbackup and high availability)
8. (Ensures data availabilityduring failures)
9. Manage multiple instanceswith additional resources
10. (Potential complexity inmanagement)

