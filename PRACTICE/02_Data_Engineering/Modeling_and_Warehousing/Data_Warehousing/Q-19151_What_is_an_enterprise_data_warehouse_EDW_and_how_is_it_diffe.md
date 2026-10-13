---
type: "practice"
difficulty: "Basic"
---

# What is an enterprise data warehouse (EDW) and how is it different from a departmental data mart?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Data Warehouse Fundamentals](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a nutshell, an Enterprise Data Warehouse (EDW) is a centralized repository that consolidates data from various sources across an organization to support decision-making processes. It differs fundamentally from a Departmental Data Mart in terms of scope, structure, and purpose.

#### Triaging the Question

The core of this question revolves around understanding the concept of EDWs and distinguishing them from departmental data marts. The interviewer is probing your grasp on key differences such as scale, data sources, and intended use cases. Assumptions include knowledge of basic data warehousing concepts like ETL processes, star/snowflake schemas, and OLAP.

#### First-Principles Solution

An Enterprise Data Warehouse (EDW) is a large-scale, centralized database designed to support the business intelligence needs of an entire organization. It integrates data from multiple sources, ensuring consistency and completeness for high-level analysis and reporting. In contrast, a Departmental Data Mart is a subset of data extracted from an EDW or other source systems tailored specifically for a particular department's needs.

Scale: An EDW typically handles vast volumes of diverse data from various departments, whereas a Departmental Data Mart focuses on specific business areas.
Scope: The scope of an EDW is broader, encompassing all business units and processes, while a Data Mart targets more localized or functional requirements.
ETL Processes: EDWs use complex ETL (Extract, Transform, Load) procedures to consolidate data from different sources, ensuring consistency across the organization. Departmental Data Marts may have simpler ETL processes if they are directly fed by transactional systems.
Purpose: The primary goal of an EDW is strategic analysis and long-term business planning, whereas departmental marts support tactical operations and specific departmental goals.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, while both EDWs and Departmental Data Marts are crucial components in the data architecture, they serve different purposes. Misunderstanding these differences can lead to poor design decisions, such as attempting to use a departmental data mart for enterprise-wide analytics or vice versa.

A common mistake is assuming that a single data warehouse (whether an EDW or a Data Mart) can replace all other sources of information within the organization. Each has its role and should be designed with clear boundaries to maximize effectiveness.

#### Follow-ups
might include:

How do you handle latency in an EDW?
What are some challenges in maintaining an EDW compared to departmental data marts?

**Comparison:**

```
EDW Features	Departmental Data Mart Features
Consolidates multiple sources	Specializes in a single area
Supports strategic analysis	Facilitates tactical operations
Uses complex ETL processes	Simpler, more direct
Larger scale	Smaller and focused
```

**Code:**

```python
# Example ETL process pseudo-code for an EDW
def edw_etl(source_systems):
    transformed_data = []
    for system in source_systems:
        raw_data = extract_from_system(system)
        cleaned_data = transform_data(raw_data)
        loaded_data = load_to_warehouse(cleaned_data)
        transformed_data.append(loaded_data)
    return transformed_data
```

**Example input / output:**

```text
Input:
EDW ETL Process with sources: ["Sales", "Marketing", "Finance"]

Output:
- Transformed data from Sales, Marketing, and Finance stored in Hadoop.
- Data cleaned and transformed to meet enterprise-wide standards.
```

**Data:**

```json
{
  "EDW": {
    "DataSources": ["Sales", "Marketing", "Finance"],
    "Transformations": [
      {"source": "Sales", "target": "CustomerID"},
      {"source": "Marketing", "target": "CampaignId"}
    ],
    "Storage": "Hadoop",
    "Purpose": "Strategic Analysis"
  },
  "DepartmentalDataMart": {
    "DataSources": ["Sales"],
    "Transformations": [
      {"source": "Sales", "target": "Revenue"}
    ],
    "Storage": "SQLServer",
    "Purpose": "Tactical Operations"
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Sources
4. Are sources diverse?
5. EDW
6. Departmental Data Mart
7. Consolidate, transform, storein Hadoop
8. Extract, transform, store inSQLServer

