---
type: "practice"
difficulty: "Basic"
---

# What does 'subject-oriented' mean when describing a data warehouse?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Kimball Approach](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Kimball_Approach.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Subject-oriented in data warehousing means organizing and structuring data around specific subjects or business areas, rather than the operational transactions that generate them. This approach ensures that all data related to a particular subject is easily accessible and consistently stored together.

#### Triaging the Question

The question asks us to explain what "subject-oriented" means when describing a data warehouse. The interviewer might be probing your understanding of how data is structured in a data warehouse, particularly the concept of organizing it around subjects or themes. To answer this effectively, we need to define subject-oriented design and highlight its benefits.

#### First-Principles Solution

Definition: Subject-oriented design involves creating tables (or schemas) that focus on specific business areas such as sales, customers, inventory, etc. Each table contains only data relevant to a particular subject.

#### Example
In a retail company’s data warehouse, there might be separate fact and dimension tables for each domain like "Sales," "Customer," "Product," and "Time." This approach makes querying and analyzing specific business areas more straightforward.

Benefits:

Simplicity in Queries: Queries are simpler because they only need to include the relevant tables.
Improved Performance: Join operations between related fact and dimension tables are efficient since data is already well-organized.
Maintainability: Easier to maintain and update as changes can be made within a specific subject area.

Comparison with Other Paradigms:

Time-Oriented Design: This focuses on different time periods, such as daily or monthly snapshots. While useful for trend analysis, it doesn’t focus on business areas.
Entity-Oriented Design: Organizes data around entities (e.g., customers) without focusing on specific subjects.

#### Closing Summary
and Common Mistakes/Follow-Ups

Subject-oriented design is crucial in data warehousing as it aligns with the way businesses analyze their data. It simplifies querying, improves performance, and enhances maintainability. A common mistake is confusing subject-oriented design with other paradigms like time or entity orientation.

If asked for an example, you might discuss a retail company’s data warehouse where there are separate schemas for sales, customers, products, etc., making it easier to query specific business areas such as customer churn analysis or product performance.

**Comparison:**

```
Orders
Sales
CustomerDemographics
```

**Code:**

```text
# Example SQL schema
CREATE SCHEMA Sales;
CREATE SCHEMA Customers;

USE Sales;

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    ProductID INT,
    Quantity INT,
    Price DECIMAL(10, 2),
    OrderDate DATE
);

USE Customers;

CREATE TABLE CustomerDemographics (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100)
);
```

**Example input / output:**

```text
Input:
- Query to find all orders made by customers in the "Sales" schema

Output:
+------------+-----------+----------+--------+-------+------------+
| OrderID    | CustomerID| ProductID| Quantity| Price  | OrderDate  |
+------------+-----------+----------+--------+-------+------------+
|          1 |        101|       201|       3 | 49.99 | 2023-01-05 |
|          2 |        102|       202|       2 | 69.99 | 2023-01-07 |
+------------+-----------+----------+--------+-------+------------+
```

**Data:**

```json
{
  "Sales": {
    "Orders": [
      {"OrderID": 1, "CustomerID": 101, "ProductID": 201, "Quantity": 3, "Price": 49.99, "OrderDate": "2023-01-05"},
      {"OrderID": 2, "CustomerID": 102, "ProductID": 202, "Quantity": 2, "Price": 69.99, "OrderDate": "2023-01-07"}
    ]
  },
  "Customers": {
    "CustomerDemographics": [
      {"CustomerID": 101, "FirstName": "John", "LastName": "Doe", "Email": "john.doe@example.com"},
      {"CustomerID": 102, "FirstName": "Jane", "LastName": "Smith", "Email": "jane.smith@example.com"}
    ]
  }
}
```

**Flow:**

1. "Subject-Oriented Schema"
2. "Query"
3. "Data Engineer"
4. "Subject-Oriented Schema"
5. "Query"
6. "Data Engineer"
7. Design schema around subjects (e.g., Sales, Customers)
8. Return relevant data based on subject query

