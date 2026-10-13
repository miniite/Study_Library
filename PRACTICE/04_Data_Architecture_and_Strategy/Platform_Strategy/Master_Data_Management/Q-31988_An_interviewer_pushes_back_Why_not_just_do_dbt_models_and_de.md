---
type: "practice"
difficulty: "Deep"
---

# An interviewer pushes back: 'Why not just do dbt models and dedupe in the warehouse — why buy MDM at all?' How do you respond and where does that approach actually break?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-domain MDM Programs](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Multi-domain_MDM_Programs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Master Data Management (MDM) goes beyond basic database transformation tasks like dbt models and deduplication in the warehouse. It's a strategic framework to ensure data integrity, consistency, and accuracy across multiple systems, which is crucial for organizations that need robust data governance and real-time decision-making.

The interviewer is asking about the limitations of using only ETL tools (like dbt) and data warehousing techniques for deduplication. While these methods are effective for some use cases, they often fall short in handling complex business requirements and ensuring comprehensive data quality across an enterprise.

Here’s a step-by-step first-principles approach to address this:

Purpose of MDM: MDM is designed to manage master data—data that defines the core entities of your organization (e.g., customers, products, employees). It ensures these critical pieces of information are accurate and consistent across all systems.

Data Quality Across Multiple Sources: While dbt can handle ETL processes and deduplication within a single warehouse, MDM tools typically integrate with multiple disparate sources, ensuring that changes in any source system are reflected consistently in all downstream systems.

Complex Deduplication Logic: The logic for deduplicating master data is often more complex than simple field comparisons. MDM tools allow for sophisticated rules and policies to handle situations like partial matches, changing attributes over time, and historical data versions.

Governance and Compliance: MDM solutions provide robust governance and compliance features that go beyond basic data cleaning. They can track lineage of data changes, maintain audit trails, and apply complex validation rules to ensure regulatory compliance.

Real-Time Data Synchronization: In real-time or near-real-time scenarios, where multiple systems need immediate updates, MDM tools are better equipped to handle the complexities of continuous synchronization compared to batch processes in dbt.

Integration with Other Systems: MDM solutions often integrate more seamlessly with other enterprise applications and data platforms, providing a centralized point for data management that can be queried or updated by various stakeholders.

Now, let's break down where using just dbt and deduplication in the warehouse might fall short:

Limited Scope: dbt models are typically designed to work within the context of a specific analytics project or dataset. They may not cover all necessary data sources and business processes.

Inadequate Rules for Deduplication: While you can implement some deduplication logic in dbt, it's often less flexible and comprehensive compared to dedicated MDM tools, which support complex rules and workflows.

Lack of Governance: dbt and data warehouse deduplication don’t provide the same level of governance as MDM solutions. There are no built-in features for tracking lineage, ensuring compliance, or managing historical data versions.

In summary, while ETL and dbt can handle some aspects of data management within a single environment, they lack the breadth and depth needed to manage master data effectively across an enterprise. MDM is essential for maintaining high-quality, consistent, and compliant data that spans multiple systems and business processes.

**Comparison:**

| ID | Name | Email                |
|----|------|----------------------|
| 1  | Alice| alice@example.com    |
| 2  | Bob  | bob@example.com      |
| 3  | Alice| alice@another.com    |

**Code:**

```text
# Example MDM Rules in Python
def deduplicate_customers(customer_data):
    # Complex deduplication logic using fuzzy matching
    pass

def validate_customer(customer_data):
    # Validation rules for customer attributes
    pass
```

**Example input / output:**

```text
# Example Input-Output for MDM Deduplication Logic
customer_data = {
    "customers": [
        {"id": 1, "name": "Alice", "email": "alice@example.com"},
        {"id": 2, "name": "Bob", "email": "bob@example.com"},
        {"id": 3, "name": "Alice", "email": "alice@another.com"}
    ]
}

deduplicated_customers = deduplicate_customers(customer_data)
print(deduplicated_customers)  # Output: [1, 2] - Alice is duplicated
```

**Data:**

```json
{
  "customers": [
    {"id": 1, "name": "Alice", "email": "alice@example.com"},
    {"id": 2, "name": "Bob", "email": "bob@example.com"},
    {"id": 3, "name": "Alice", "email": "alice@another.com"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Data Source
4. Is Data Consistent?
5. Deduplicate and Validate
6. Store in Warehouse
7. MMD Rule Engine
8. Consistent Data AcrossSystems

