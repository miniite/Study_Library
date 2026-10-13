---
type: "practice"
difficulty: "Intermediate"
---

# How do you document a dimensional model so analysts and other engineers never misuse the grain or additivity, and where does that documentation live and stay current?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Fact Table Grain](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_Table_Grain.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that analysts and data engineers understand and correctly use a dimensional model, particularly in terms of grain and additivity, we need to establish robust documentation practices. This involves defining the dimensions and facts with clarity, specifying their granularity, and ensuring these details are well-documented and easily accessible.

#### Triaging the Question

The question is probing how to effectively document a dimensional model, specifically focusing on:

Grain: The level of detail at which data is stored.
Additivity: Whether aggregates can be calculated across all levels of granularity.

We need to ensure that this documentation is comprehensive and accessible, so it remains relevant as the schema evolves over time. Assumptions include:

The use of either Star or Snowflake schema for dimensional modeling.
Collaboration among data engineers, analysts, and other stakeholders.

#### First Principles Solution
Define the Dimensions and Facts: Clearly outline each dimension (e.g., Date, Customer) and fact table (e.g., Sales). Specify their attributes and granularities.
Document Grain and Additivity: For each measure in a fact table, document its additivity (e.g., Sum of sales is additive over time).
Version Control: Use version control to track changes in the schema and documentation.
Automated Documentation Tools: Employ tools that generate documentation from database metadata.
Step-by-Step Solution
Identify Dimensions and Facts:

#### Example
For a Sales fact table, dimensions include Date, Customer ID, Product Category.
Document Grain:
Each dimension should specify its grain (e.g., Date -> Day, Week).
Define Additivity:
Document which measures can be aggregated across different granularities (e.g., Sum of Sales is additive over days but not necessarily months).

#### Example
Documentation
Sales Fact Table
Date -- Day
Customer ID -- Customer
Product Category -- Category
Sales Amount -- Sum isadditive across all granularitiesof Date, Customer, andProduct Category

#### Implementation
Example
Document Store:
Use a dedicated repository (e.g., Confluence) for storing documentation.
Version Control:
YAML
---
version: 1.0
schema_changes:
  - timestamp: 2023-09-01
    changes:
      - fact_table: Sales
        dimensions_added: [Product Category]
      - dimension: Date
        granularity_changed: Day -> Week
Automated Tools:
Use tools like DBT (Data Build Tool) to generate documentation from database metadata.

#### Closing Summary
and Common Mistakes/Follow-Ups

The key is to ensure that the dimensional model documentation is clear, accessible, and up-to-date. Common mistakes include neglecting to document additivity or allowing the schema to evolve without updating documentation. Follow-ups could involve discussing specific tools for automation and version control, as well as strategies for ensuring continuous maintenance of the documentation.

**Code:**

```yaml
---
version: 1.0
schema_changes:
  - timestamp: 2023-09-01
    changes:
      - fact_table: Sales
        dimensions_added: [Product Category]
      - dimension: Date
        granularity_changed: Day -> Week
```

