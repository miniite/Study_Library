---
type: "practice"
difficulty: "Expert"
---

# What lineage guarantees would you require end to end across a mesh, and how do you assemble lineage when each domain owns its own tooling?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Ownership vs Cross-cutting Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Domain_Ownership_vs_Cross-cutting_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Data Mesh architecture, ensuring end-to-end data lineage is crucial for maintaining trust and accountability in your data ecosystem. As an architect/principal Data Engineer, my focus would be on establishing comprehensive lineage guarantees that cover all domains while leveraging their unique tooling. The key to assembling this lineage lies in standardizing metadata ingestion processes and utilizing a central lineage store.

The goal is to create a unified view of the data journey from raw sources through transformation steps and finally to the insights they support, regardless of which domain owns the specific tools or processes involved.

#### Triage
What's Being Probed

The core question here involves understanding how lineage can be constructed in a distributed environment where different domains use their own tooling. The interviewer is probing your ability to design a robust metadata strategy that bridges these silos and ensures traceability across the entire data mesh.

#### Assumptions
Each domain has its own tools for processing, transforming, or ingesting data.
There is a need for central management of lineage metadata to ensure consistency and accountability.
The domains may not share their internal tooling standards but can adhere to agreed-upon metadata formats.

#### First Principles Solution
Metadata Standardization: Define common metadata fields such as data_source, processing_step, transformer, output_location, etc., that will be used consistently across all domains.
Lineage Store: Implement a central lineage store (e.g., a NoSQL database, graph database) where each domain can push their lineage information in the agreed-upon format.
Ingestion Mechanism: Develop standardized APIs or scripts that allow each domain to automatically or semi-automatically capture and report their lineage data into the central store.
Data Provenance Tracking: Ensure that every piece of transformed data has a clear provenance, linking it back to its original source and all intermediate steps.
Automated Validation: Use automated tools to validate the consistency and completeness of the lineage metadata reported by each domain.
Assembly Process

Domain-Specific Metadata Ingestion:

Each domain would write their own scripts or use existing tooling to capture lineage information for their processes.
For example, if a data scientist uses Python, they might write a script that logs the steps and parameters used in data transformations.

Standardized Output Format:

The metadata captured by each domain should follow a predefined format (e.g., JSON) that includes fields like timestamp, processing_step, input_data_sources, output_location, etc.

#### Example
JSON
{
  "domain": "Sales",
  "data_source": ["Sales_DB"],
  "processing_step": "Data Cleaning and Transformation",
  "parameters": {"column_renamed": true, "missing_value_fill": "mean"},
  "output_location": "/cleaned_sales_data.csv"
}

Central Lineage Store:

The central lineage store would aggregate these metadata entries from all domains.
It could be a graph database like Neo4j or a relational database with a schema designed to support lineage queries.

#### Example
Workflow
Suppose the Sales domain processes data and pushes its lineage information into the central store. The E-commerce domain then uses this data in their own analytics, capturing additional steps but linking back to the initial transformation.
Lineage query: Find all data from Sales domain that was used in E-commerce analytics after 2023-10-01.

#### Closing Summary

End-to-end lineage guarantees in a Data Mesh require standardization of metadata across domains and a central lineage store to consolidate this information. By ensuring each domain contributes its lineage metadata consistently, we can build a comprehensive view of the data's journey.

#### Common Mistakes/Follow-Ups
Failing to Standardize Metadata: Lack of standardization leads to inconsistencies that can make it difficult to trace data lineage.
Insufficient Automation: Relying too much on manual processes increases the risk of errors and reduces the accuracy of lineage information.
Limited Visibility Across Domains: Ensuring all domains are aware of and contribute to the central lineage store is crucial.

Would you like me to elaborate on any specific aspect, such as the technical implementation details or potential challenges in scaling this approach?

**Comparison:**

| Domain | Data Source | Processing Step                      | Parameters                | Output Location        |
|--------|-------------|--------------------------------------|---------------------------|-----------------------|
| Sales  | Sales_DB    | Data Cleaning and Transformation     | column_renamed: True      | /cleaned_sales_data.csv|
| E-commerce | /cleaned_sales_data.csv   | Aggregation for Analytics            | aggregation_method: sum, time_window: monthly | /sales_analytics.csv |

**Code:**

```python
# Example Python script for metadata ingestion
import json

def log_metadata(data_source, processing_step, parameters, output_location):
    metadata = {
        "domain": "Sales",
        "data_source": data_source,
        "processing_step": processing_step,
        "parameters": parameters,
        "output_location": output_location
    }
    with open("lineage_data.json", "a") as f:
        json.dump(metadata, f)
        f.write("\n")

log_metadata(["Sales_DB"], "Data Cleaning and Transformation", {"column_renamed": True, "missing_value_fill": "mean"}, "/cleaned_sales_data.csv")
```

**Example input / output:**

```text
Input:
- Sales domain processes data using Python script and logs metadata to a JSON file.
- E-commerce domain uses this processed data, captures additional steps in its own process.

Output:
Central lineage store aggregates metadata from both domains. Example query:
"Find all data from Sales domain that was used in E-commerce analytics after 2023-10-01."

Result:
```

**Data:**

```json
[
  {
    "domain": "Sales",
    "data_source": ["Sales_DB"],
    "processing_step": "Data Cleaning and Transformation",
    "parameters": {"column_renamed": true, "missing_value_fill": "mean"},
    "output_location": "/cleaned_sales_data.csv"
  },
  {
    "domain": "E-commerce",
    "data_source": ["/cleaned_sales_data.csv"],
    "processing_step": "Aggregation for Analytics",
    "parameters": {"aggregation_method": "sum", "time_window": "monthly"},
    "output_location": "/sales_analytics.csv"
  }
]
```

**Flow:**

1. Data Source
2. Domain-specific Processing
3. Sales_domain
4. E-commerce_domain
5. Cleaned Data
6. Aggregated Analytics
7. Central Lineage Store
8. Lineage Query
9. Trace Data Journey

