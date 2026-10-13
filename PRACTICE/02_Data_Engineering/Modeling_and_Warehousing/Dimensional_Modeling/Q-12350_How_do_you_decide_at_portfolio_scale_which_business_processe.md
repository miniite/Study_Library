---
type: "practice"
difficulty: "Deep"
---

# How do you decide, at portfolio scale, which business processes get their own fact tables versus being merged, balancing team autonomy against sprawl?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Star Schemas and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding how to structure a data model in a large-scale database environment, especially with multiple business processes involved, the key is to balance the benefits of team autonomy and modularity with the need for efficient data retrieval and maintenance. At a portfolio scale, this decision involves understanding both the technical constraints and the business requirements.

To start, let's break down what we are dealing with:

Team Autonomy: This refers to allowing teams to manage their own data models independently, which can lead to faster development cycles and better alignment with specific business needs.
Sprawl: This is the risk of having too many fact tables, leading to increased complexity in data management and query performance.

Here’s a step-by-step approach from first principles:

Identify Core Business Processes: Determine the primary business processes that drive critical decision-making across your organization. These processes will often need their own fact tables to capture detailed transactional data.
Evaluate Team Requirements: Understand what each team needs in terms of data granularity and reporting requirements. Teams with similar or overlapping needs can share fact tables, while distinct teams might have unique datasets.

#### Performance Considerations
Fact tables that are queried frequently for large datasets should be kept separate to avoid performance bottlenecks. Similarly, dimension tables (which describe attributes) should be normalized where possible but shared across relevant fact tables.
Data Integrity and Consistency: Ensure that the data model supports strong referential integrity rules to maintain consistency. Merging unrelated processes into a single fact table can compromise this.

Let's reason through these steps in more detail:

Step 1: Identify Core Business Processes

For example, if you have sales, marketing, and customer service teams, each might focus on different aspects of customer behavior. Sales data might include transactional details, while customer service data could capture interactions and feedback. These would typically be separate fact tables due to their distinct nature.

Step 2: Evaluate Team Requirements

#### Cons
ider the scale and complexity of each team’s operations:

Sales Team: Focuses on detailed sales transactions, product information, and customer demographics.
Marketing Team: Tracks campaign effectiveness, lead generation, and demographic insights.

In this case, it would be reasonable to have separate fact tables for these teams but ensure common dimensions like customers are shared across relevant tables.

Step 3: Performance Considerations

Fact tables with high cardinality (many unique values) should be kept separate. For instance, a sales fact table might contain millions of transactions, while a marketing fact table could track smaller but more frequent interactions. Combining these in one large table could degrade query performance.

Step 4: Data Integrity and Consistency

Maintaining strong data integrity rules is crucial:

Ensure that customer IDs are consistent across all relevant tables.
Use look-up tables for common attributes like product categories to enforce referential integrity.

By following these principles, you can create a robust data model that balances team autonomy with efficient data management. However, this approach requires careful planning and ongoing maintenance.

#### Closing Summary

In summary, deciding which business processes get their own fact tables versus being merged involves balancing the needs of individual teams for autonomy against the necessity of maintaining an organized, performant, and consistent database structure. Effective data modeling requires a deep understanding of both the technical requirements and the business context to ensure optimal performance and scalability.

#### Common Mistakes/Follow-Ups
Ignoring Performance: Merging unrelated processes into large fact tables can lead to poor query performance.
Over-Engineering: Creating overly detailed models with too many separate fact tables without clear justification can add complexity unnecessarily.
Neglecting Data Consistency: Inadequate management of shared dimensions and look-up tables can result in data inconsistencies.

**Comparison:**

```
Customer ID	Name	Email
1	John Doe	john@example.com
2	Jane Smith	jane@example.com
Product ID	Name
101	Product A
102	Product B
Transaction ID	Customer ID	Product ID	Quantity	Price	Sale Date
1	1	101	2	$50.00	2023-01-01
2	2	102	1	$75.00	2023-01-02
Campaign ID	Name	Start Date	End Date
1001	Winter Sale	2023-01-01	2023-01-31
1002	Spring Promotions	2023-04-01	2023-05-31
Interaction ID	Customer ID	Campaign Name	Interaction Date	Interaction Type
1	1	Winter Sale	2023-01-05	Email
2	2	Spring Promotions	2023-04-15	Social Media
```

**Code:**

```sql
-- Example schema for sales and marketing teams
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(100)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE sales_transactions (
    transaction_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10, 2),
    sale_date DATE
);

CREATE TABLE marketing_campaigns (
    campaign_id INT PRIMARY KEY,
    name VARCHAR(255),
    start_date DATE,
    end_date DATE
);

CREATE TABLE marketing_interactions (
    interaction_id INT PRIMARY KEY,
    customer_id INT,
    campaign_id INT,
    interaction_date DATE,
    interaction_type VARCHAR(50)
);
```

**Example input / output:**

```text
-- Query for sales transactions by customer in January 2023
SELECT c.name AS customer_name, s.quantity, p.name AS product_name, s.price, s.sale_date
FROM customers c
JOIN sales_transactions s ON c.customer_id = s.customer_id
JOIN products p ON s.product_id = p.product_id
WHERE EXTRACT(MONTH FROM s.sale_date) = 1 AND EXTRACT(YEAR FROM s.sale_date) = 2023;

-- Query for interactions with a specific campaign
SELECT mi.interaction_id, c.name AS customer_name, m.name AS campaign_name, mi.interaction_date, mi.interaction_type
FROM marketing_interactions mi
JOIN customers c ON mi.customer_id = c.customer_id
JOIN marketing_campaigns m ON mi.campaign_id = m.campaign_id
WHERE m.name = 'Winter Sale';
```

**Data:**

```json
{
  "customers": [
    {"customer_id": 1, "name": "John Doe", "email": "john@example.com"},
    {"customer_id": 2, "name": "Jane Smith", "email": "jane@example.com"}
  ],
  "products": [
    {"product_id": 101, "name": "Product A"},
    {"product_id": 102, "name": "Product B"}
  ],
  "sales_transactions": [
    {"transaction_id": 1, "customer_id": 1, "product_id": 101, "quantity": 2, "price": 50.00, "sale_date": "2023-01-01"},
    {"transaction_id": 2, "customer_id": 2, "product_id": 102, "quantity": 1, "price": 75.00, "sale_date": "2023-01-02"}
  ],
  "marketing_campaigns": [
    {"campaign_id": 1001, "name": "Winter Sale", "start_date": "2023-01-01", "end_date": "2023-01-31"},
    {"campaign_id": 1002, "name": "Spring Promotions", "start_date": "2023-04-01", "end_date": "2023-05-31"}
  ],
  "marketing_interactions": [
    {"interaction_id": 1, "customer_id": 1, "campaign_id": 1001, "interaction_date": "2023-01-05", "interaction_type": "Email"},
    {"interaction_id": 2, "customer_id": 2, "campaign_id": 1002, "interaction_date": "2023-04-15", "interaction_type": "Social Media"}
  ]
}
```

**Flow:**

1. Identify Core BusinessProcesses
2. Evaluate Team Requirements
3. Performance Considerations
4. Data Integrity and Consistency
5. Final Model Design

