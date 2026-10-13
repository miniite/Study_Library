---
type: "practice"
difficulty: "Basic"
---

# Why would a company end up with more than one fact table over time?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Aggregate Fact Tables](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Aggregate_Fact_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, companies often start with simple schemas and evolve to more complex ones as their business needs grow. One common evolution is moving from a single fact table model to multiple fact tables, which can arise due to various business requirements or data volume issues. This answer will cover why companies might end up with more than one fact table over time, starting from basic principles.

#### Triage
Understanding the Question

The question probes the reasons behind having multiple fact tables in a data warehouse design and how this impacts data modeling strategies. It assumes familiarity with basic concepts like star schema and snowflake schema. The key is to understand the trade-offs between simplicity and complexity, as well as the implications of scale and performance.

#### First-Principles Solution

Business Requirements and Data Variety: As a company grows, it may need to capture data from diverse sources or measure various aspects of its business operations. Each fact table typically represents one specific metric or business process. For instance, an online retail company might have separate fact tables for sales transactions (orders) and customer reviews.

Data Volume and Performance: Large volumes of data can impact query performance. A single fact table can become unwieldy as it grows, leading to slower queries and increased storage costs. By splitting the data into multiple smaller fact tables, companies can optimize for both read and write operations.

Data Granularity and Dimensionality: Different fact tables may require different levels of granularity or dimensionality. For example, a transactional fact table might include detailed product information, while another fact table focused on customer behavior might have less granular data but more dimensions related to marketing campaigns.

Historical Data Management: Some business processes require the retention of historical data for analysis over longer periods. Separate fact tables can help manage this by storing different types of historical data in appropriate time frames.

Complex Business Logic and Compliance: Certain businesses might have complex rules or regulatory requirements that necessitate separate fact tables to ensure compliance and maintain data integrity.

Data Warehouse Design Patterns: Advanced patterns like snowflake schema introduce intermediate dimension tables, which can further decompose the model into multiple fact tables for better organizational clarity and reduced redundancy.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, having more than one fact table in a data warehouse is often driven by business needs such as data variety, performance optimization, and compliance requirements. Each fact table serves a specific purpose, making the overall model more flexible and scalable.

#### Common mistakes
include overcomplicating the schema without clear business justification or ignoring performance implications. Follow-up questions might explore the trade-offs between different design choices, how to balance ease of maintenance with query complexity, or examples from real-world data warehousing projects.

**Comparison:**

```
FLOWCHART · MERMAID
erDiagram
    CUSTOMER ||--o{ SALES_TRANSACTION : makes
    PRODUCT ||--o{ SALES_TRANSACTION : sold
    CUSTOMER }|--X{ REVIEW : wrote
    PRODUCT }|--X{ REVIEW : rated
```

**Code:**

```sql
-- Example of a simple star schema fact table creation
CREATE TABLE sales_transactions (
    transaction_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT,
    quantity INT,
    amount DECIMAL(10, 2),
    sale_date DATE
);
```

**Example input / output:**

| transaction_id | product_id | customer_id | quantity | amount |
|----------------|------------|-------------|----------|--------|
| 1              | 101        | 1001        | 5        | 49.95  |
| 2              | 102        | 1002        | 8        | 79.95  |
| 3              | 103        | 1003        | 6        | 59.95  |

**Data:**

```json
{
  "sales_transactions": {
    "transaction_id": [1, 2, 3],
    "product_id": [101, 102, 103],
    "customer_id": [1001, 1002, 1003],
    "quantity": [5, 8, 6],
    "amount": [49.95, 79.95, 59.95],
    "sale_date": ["2023-01-01", "2023-01-02", "2023-01-03"]
  }
}
```

**Flow:**

1. Sales Transaction Fact Table
2. (Product)
3. (Customer)
4. Review Fact Table
5. (Customer)
6. (Product)

