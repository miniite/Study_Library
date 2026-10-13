---
type: "practice"
difficulty: "Deep"
---

# How do you decide whether to fan out one source into many marts or build one wide model, given query patterns and maintenance cost?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ ETL vs ELT Design](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/ETL_vs_ELT_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between fanning out one source into multiple data marts versus building a single wide model, it's crucial to consider both the query patterns required by your business and the maintenance costs associated with each approach. This decision is fundamentally about balancing performance, flexibility, and operational overhead.

First, let’s triage what the question is really asking:

Query Patterns: Are they complex, involving joins across multiple tables? Or are they simple, focusing on a single data mart?
Maintenance Cost: How much effort will be required to manage and update each model?

#### First-Principles Solution
1. Query Complexity

If the query patterns involve complex joins or require a broad range of data from different sources, fanning out into multiple marts might be more suitable. Each mart can focus on specific business needs, making queries simpler and faster.

For example, consider an e-commerce company:

Sales Mart: Contains transactional data for sales reports.
Marketing Mart: Focuses on customer behavior and marketing effectiveness.
Inventory Mart: Tracks stock levels and supply chain operations.

Each of these marts can be optimized for specific queries, reducing the complexity of joins required by query designers. This approach is more maintainable as changes in one mart do not necessarily impact others.

2. Maintenance Cost

Building a single wide model requires more initial effort to normalize and de-normalize data appropriately across various business perspectives. However, it can reduce maintenance costs over time if the structure remains stable and the queries are straightforward.

#### Cons
ider an insurance company:

Wide Model: Contains all the necessary fields for different reports (claims, policy management, risk assessment).

In this case, maintaining a single model ensures that changes to data fields or relationships have only one place to be updated. However, initial setup can be complex due to the need for thorough normalization and denormalization strategies.

3. Trade-offs

The decision depends on the trade-off between query performance and maintenance:

Fan-out Approach: Easier query design but may lead to more maintenance work if multiple marts are involved.
Wide Model: Harder initial setup but potentially lower ongoing maintenance cost with stable queries.

#### Example Scenario

Let’s say we have an online retail company that needs real-time analytics for sales, marketing campaigns, and customer service. Here, a fan-out approach might be more appropriate:

Sales Mart: Real-time transactional data.
Marketing Mart: Campaign performance metrics.
Customer Service Mart: Customer support interactions.

This setup allows each team to focus on their specific queries without overwhelming complexity in the ETL processes.

#### Closing Summary
and Common Mistakes
Mistake 1: Assuming a single wide model is always better because it reduces maintenance costs. This can lead to overly complex data models that are hard to maintain.
Mistake 2: Fanning out too much, leading to a proliferation of marts without clear ownership and maintenance plans.

#### Conclusion

The choice between fanning out into multiple marts or building a single wide model is deeply tied to the nature of query patterns and the level of maintenance required. A balanced approach that considers both performance and maintainability will lead to the best outcome for your organization.

**Comparison:**

```
2023-09-01
2023-09-03
2023-09-05
2023-09-07
2023-09-09
2023-09-11
2023-09-13
2023-09-15
2023-09-17
2023-09-19
Sales Mart
Single Model
Marketing Mart
Customer Service Mart
Fan-out Approach
Wide Model
Fan-out Approach vs. Wide Model
```

**Code:**

```text
# Example ETL Pipeline (Fan-out Approach)
data_source = read_data('source_table')
sales_mart = process_sales_data(data_source)
marketing_mart = process_marketing_data(data_source)
customer_service_mart = process_customer_service_data(data_source)

save_to_marts(sales_mart, 'sales_mart')
save_to_marts(marketing_mart, 'marketing_mart')
save_to_marts(customer_service_mart, 'customer_service_mart')
```

**Example input / output:**

```text
# Input Data (Fan-out Approach)
Input: {"id": "12345", "customer_id": "A001", "product_id": "P001", "quantity_sold": 2, "sale_date": "2023-09-01"}

Output:
Sales Mart: {"id": "12345", "customer_id": "A001", "product_id": "P001", "quantity_sold": 2, "sale_date": "2023-09-01"}
Marketing Mart: {"campaign_id": null, "campaign_name": null, "lead_source": null, "conversion_rate": null}
Customer Service Mart: {"customer_id": "A001", "support_ticket_id": null, "issue_description": null, "resolution_date": null}

# Input Data (Wide Model Approach)
Input: {"id": "67890", "sale_data": [{"product_id": "P002", "quantity_sold": 5, "sale_date": "2023-09-10"}],
        "campaign_data": [{"campaign_id": "C001", "campaign_name": "BackToSchool2023", "lead_source": "Email"},
                          {"campaign_id": "C002", "campaign_name": "OnlinePromo2023", "lead_source": "SocialMedia"}],
        "customer_service_data": [{"customer_id": "B001", "issue_description": "Refund Requested"}]}

Output: {"id": "67890", "sale_data": [{"product_id": "P002", "quantity_sold": 5, "sale_date": "2023-09-10"}],
         "campaign_data": [{"campaign_id": "C001", "campaign_name": "BackToSchool2023", "lead_source": "Email"},
                           {"campaign_id": "C002", "campaign_name": "OnlinePromo2023", "lead_source": "SocialMedia"}],
         "customer_service_data": [{"customer_id": "B001", "issue_description": "Refund Requested"}]}
```

**Data:**

```yaml
# Data Model Schema - Fan-out Approach
sales_mart:
  fields: [id, customer_id, product_id, quantity_sold, sale_date]
marketing_mart:
  fields: [campaign_id, campaign_name, lead_source, conversion_rate]
customer_service_mart:
  fields: [customer_id, support_ticket_id, issue_description, resolution_date]
```

**Flow:**

1. Query Pattern Complexity
2. FAN-OUT
3. Multiple Marts per BusinessPerspective
4. Maintainability
5. Wide Model
6. SINGLE MODEL
7. Single, Comprehensive DataStructure
8. Maintenance Overhead
9. Decision Criteria

