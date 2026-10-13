---
type: "practice"
difficulty: "Expert"
---

# Compare OpenMetadata, DataHub, Amundsen, and a commercial catalog on their metadata models and extensibility. If you were standardizing across an org, which would you pick and why — or would you build?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cataloging External Data](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Cataloging_External_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating open-source data catalogs like OpenMetadata, DataHub, Amundsen, and commercial solutions for metadata management, it's crucial to consider several factors including extensibility, metadata model support, ease of integration, community support, and total cost of ownership. Let’s break down each option based on these criteria.

Firstly, OpenMetadata is a modern, open-source data catalog designed with flexibility in mind. It supports a wide array of connectors for different data sources and has a modular architecture that allows for easy extension through plugins. Its metadata model is relatively flexible but may require more configuration to fully align with specific organizational needs.

Next, DataHub is another robust, open-source solution that emphasizes rich metadata and lineage capabilities. It uses the Atlas schema registry from Cloudera, which provides a strong foundation for metadata management. DataHub’s extensibility through plugins is excellent, although it might have a slightly steeper learning curve compared to other solutions.

Amundsen, initially developed by Meta (formerly Facebook), offers comprehensive documentation and lineage capabilities but has faced some challenges with its evolving development status. It supports various data sources via plugins and has a strong community around it, which can be beneficial for support and contributions.

Finally, commercial catalogs like Informatica’s Data Catalog or Informatica Cloud offer enterprise-grade features such as advanced governance, enhanced security, and integration capabilities with existing infrastructure. They typically come with comprehensive documentation, dedicated support teams, and professional services.

If standardizing across an organization, my recommendation would be to build a custom solution using OpenMetadata. Here’s why:

Modularity & Flexibility: OpenMetadata's modular architecture allows for seamless integration with various data sources and tools. This can lead to a more cohesive information management system when combined with existing technologies.
Community Support & Extensibility: The active community around OpenMetadata provides ongoing support, new features, and customizations that commercial solutions may not always offer out-of-the-box.
Cost-Effectiveness: Building on OpenMetadata can be significantly cheaper than deploying a full-featured commercial solution, especially for large organizations with diverse data landscapes.

However, if the organization has stringent requirements or prefers a turnkey solution with enterprise support and features, a commercial catalog might be the better choice. In such cases, conducting a thorough evaluation to match specific needs against available solutions is crucial.

**Comparison:**

```
Tool	Modularity	Community Support	Extensibility	Cost-Effectiveness
OpenMetadata	High	High	High	Low
DataHub	Medium	Medium	High	Medium
Amundsen	Low	Low	High	Medium
Commercial	Low	Low	Low (custom)	High
```

**Code:**

```python
# Example of extending OpenMetadata
from metadata.generated.schema.entity.data.table import Table

class CustomTable(Table):
    additional_custom_field = String()

# Adding this custom table definition allows for more flexibility in metadata models.
```

**Example input / output:**

```text
Input:
- Data sources: AWS S3, Google BigQuery
- Requirements: Detailed lineage, access control

Output:
- OpenMetadata configuration with custom schema and plugins for data source connectors
- Amundsen integration for rich metadata management and community support
- Commercial catalog evaluation based on specific organizational needs
```

**Data:**

```json
{
  "data_source": "AWS S3",
  "table_name": "customer_orders",
  "columns": [
    { "name": "order_id", "type": "INT" },
    { "name": "customer_id", "type": "VARCHAR" }
  ],
  "metadata_model": {
    "schema": "custom_schema",
    "version": "1.0"
  }
}
```

**Flow:**

1. OpenMetadata
2. Customizable Metadata Model
3. Strong Community Support
4. Flexible Architecture forExtensibility
5. Cost-Effective

