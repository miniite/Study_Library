---
type: "practice"
difficulty: "Expert"
---

# How do you cap the sprawl of one-off Gold marts that each team spins up, while still giving domains autonomy? What's your platform-vs-freedom balance?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Controlling Gold Sprawl](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Controlling_Gold_Sprawl.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the sprawling issue of individual teams spinning up their own Gold Mart databases while maintaining domain autonomy, we need to establish a robust and scalable Medallion Architecture that balances platform control with team flexibility. The key idea is to centralize data curation and governance without stifling innovation and agility.

The question probes how to manage decentralized data pipelines efficiently while ensuring consistency, quality, and compliance across the organization. We'll assume the company has diverse domains (e.g., marketing, finance, sales) that each require their own Gold Mart databases for detailed analysis. The challenge lies in providing these teams with autonomy while centralizing governance.

Let's break down the solution step-by-step:

Define a Hierarchical Data Architecture:

Bronze Layer: Raw data from sources.
Silver Layer: Clean, curated, and enriched data ready for reporting.
Gold Layer: Highly aggregated and detailed analysis data tailored to specific use cases.

Centralized Data Platform vs. Domain Autonomy:

Centralize the Silver layer to ensure consistency and reduce redundancy.
Provide flexibility in the Gold layer by allowing domains to create their own databases while adhering to central guidelines.

Implement a Medallion Architecture:

Use a combination of Bronze, Silver, and Gold layers to manage data at different levels of detail.
Ensure that each domain can query both Silver and Gold layers as needed.

Platform Control Mechanisms:

Define standard schemas and data models for the Silver layer.
Implement governance rules and metadata management tools to enforce consistency across domains.
Provide a central API or microservices to interact with the Silver layer, ensuring that each domain can leverage this centralized data without duplicating effort.

Domain Freedom Mechanisms:

Allow domains to create their own Gold Mart databases by providing templates and guidelines.
Offer tools for self-service querying and reporting, reducing the need for custom development.

Monitoring and Compliance:

Implement monitoring and alerting mechanisms to ensure data quality and integrity in both Silver and Gold layers.
Use governance tools to enforce compliance with regulations and internal policies.

Now, let's summarize this approach:

By centralizing the Silver layer while providing flexibility in the Gold layer, we can achieve a balance between platform control and domain autonomy.
This strategy ensures that data is consistently curated, reducing redundancy and improving overall efficiency.
The use of standard schemas and governance tools maintains high-quality data across all domains.

#### Common mistakes
include:

Over-centralization leading to reduced agility.
Under-governance causing inconsistencies and compliance issues.
Lack of clear guidelines for domain autonomy, leading to suboptimal data practices.

In conclusion, the key is to design a balanced architecture that leverages central control while empowering individual domains. This approach ensures both consistency and flexibility, setting a foundation for scalable and maintainable data management in large organizations.

**Comparison:**

```
Layer	Purpose	Governance Rules
Bronze	Raw data storage	None
Silver	Data curation	Enforce schema and integrity rules
Gold	Detailed analysis	Use templates, follow guidelines
```

**Code:**

```python
# Example of a simple governance function in Python
def enforce_data_quality(data):
    if not validate_schema(data):
        raise ValueError("Data does not conform to the schema")
    if not check_integrity(data):
        raise ValueError("Data integrity issues detected")
```

**Example input / output:**

```
Input: A request to query the Silver layer for customer data.
Output: A clean, enriched dataset ready for reporting.
JSON
Copy
{
  "query": "SELECT * FROM customers",
  "response": {
    "data": [
      {"id": 1, "name": "John Doe", "email": "johndoe@example.com"},
      {"id": 2, "name": "Jane Smith", "email": "janesmith@example.com"}
    ],
    "metadata": {
      "table_name": "customers",
      "schema_version": "1.0"
    }
  }
}
```

**Data:**

```yaml
bronze_layer:
  - source: database1
  - source: file1.csv

silver_layer:
  - table: customers
  - table: orders
gold_layer:
  - mart: marketing_analysis
  - mart: sales_insights
```

**Flow:**

1. Data Source
2. Bronze Layer
3. Silver Layer
4. Gold Layer
5. Domain Database
6. Central API/Governance Tools

