---
type: "practice"
difficulty: "Expert"
---

# Build-vs-buy: a vendor offers an automated Data Vault generation tool. How do you evaluate whether it earns its lock-in versus hand-rolled dbt macros?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Vault](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Evaluating whether to use an automated Data Vault (DV) generation tool or rolling your own with dbt macros involves weighing several factors, including ease of implementation, maintainability, flexibility, and long-term cost. The key is to understand the trade-offs between automation and customization.

The question revolves around assessing the value proposition of a vendor-provided tool versus building it in-house using dbt macros. We need to consider the current state of our data infrastructure, the complexity of DV implementation, potential maintenance costs, and whether the tool can adapt to future changes or organizational needs.

#### Assumptions

The automated Data Vault generation tool is from a reputable vendor.
Our in-house team has experience with dbt macros and data modeling best practices.
We are using an ETL/ELT framework like dbt.

Step-by-Step Solution:

Understand the Current State of the Data Warehouse:

Assess the current complexity of our data model and whether it aligns with DV principles (de-normalization, atomicity, event sourcing).
Evaluate the existing ETL/ELT processes and tools.

Evaluate the Vendor Tool:

Review the tool’s features, such as its support for incremental loading, schema-on-read, and data quality checks.
Check if it integrates well with our current tech stack (e.g., Snowflake, Redshift).

Implement a Proof of Concept (PoC):

Set up a small-scale PoC to test the tool’s performance and reliability.
Monitor its ability to handle various data scenarios, including different types of joins and transformations.

Assess Customization Needs:

Determine if we need any customization for specific business requirements that the tool doesn’t natively support.

#### Cons
ider how much effort it would take to integrate such customizations into the tool versus building them from scratch with dbt.

Compare Long-Term Costs and Maintenance:

Calculate the total cost of ownership (TCO) for both options, considering initial setup, ongoing maintenance, and potential hidden costs.

#### Cons
ider any vendor lock-in concerns and the availability of open-source alternatives.

Evaluate Flexibility and Scalability:

Assess whether the tool can be easily adapted to new data sources or evolving business needs.
Compare its flexibility with dbt macros, which offer more control over data transformations but require ongoing development efforts.

#### Cons
ider Training and Support Needs:

Determine if using the vendor tool requires additional training for our team.
Evaluate the level of support provided by the vendor versus in-house expertise.

Decision Matrix:

Create a decision matrix to weigh the pros and cons of each option, including factors like implementation time, maintainability, and cost.
Use this matrix to make an informed decision based on our specific organizational context.

#### Closing Summary
The evaluation of whether to use a vendor-provided Data Vault generation tool or build it in-house with dbt macros is a strategic decision that involves balancing automation benefits with the need for flexibility and customization. A thorough analysis, including a PoC, cost assessment, and consideration of long-term maintenance needs, will help us make an informed choice.

#### Common Mistakes/Follow-Ups

Overestimating the ease of integration or underestimating customizations needed.
Failing to account for potential vendor lock-in or lack of open-source alternatives.
Ignoring ongoing support and training requirements when using a third-party tool.

**Comparison:**

```
Schema	Table	Description
raw	sales_transactions	Source sales data
transformed_data	Transformed sales transactions for analysis
```

**Code:**

```python
# Example of dbt macro for simple data transformation
{{ config(materialized='table') }}

with source as (
    select * from {{ ref('raw_data_source') }}
),

transformed as (
    select
        id,
        customer_name,
        transaction_date,
        amount
    from source
)

select * from transformed
```

**Example input / output:**

```text
Input:
- Raw sales transactions from a CSV file.

Output:
- A transformed dataset with relevant columns (id, customer_name, transaction_date, amount) ready for analysis.
```

**Data:**

```yaml
# Example of a simple dbt project configuration file
name: 'my_dbt_project'
version: '0.1'
config-version: 2

# models
models:
  - name: raw_data_source
    description: Raw data source table
  - name: transformed_data
    description: Transformed data table for analysis

# sources
sources:
  - name: my_raw_data
    schema: 'raw'
    tables:
      - name: sales_transactions
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Current Data ModelComplexity
6. Is DV Suitable?
7. Vendor Tool Evaluation
8. Build In-House with dbtMacros
9. PoC Setup
10. Customizations Needed?
11. Customization Effort vs.Vendor Tool Integration
12. Vendor Tool Implementation
13. In-house Development
14. Vendor Tool Adoption

