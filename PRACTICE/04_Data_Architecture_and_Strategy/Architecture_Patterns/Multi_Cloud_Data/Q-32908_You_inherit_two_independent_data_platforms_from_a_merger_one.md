---
type: "practice"
difficulty: "Expert"
---

# You inherit two independent data platforms from a merger — one on Azure Synapse and one on Snowflake-on-AWS. Lay out a 24-month convergence strategy and the sequencing of what you unify first.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-cloud Data Contracts](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When inheriting two independent data platforms, one on Azure Synapse and another on Snowflake-on-AWS, our goal is to converge them into a unified architecture over 24 months. The key is to focus on commonalities while addressing differences in the cloud environments, SQL dialects, governance policies, and cost structures. We'll start by assessing the current state of both platforms, then prioritize unification based on business criticality and technical feasibility.

#### Triaging the Question

The interviewer is probing our understanding of:

Assessing Current State: Identifying key differences and commonalities.
Pricing and Cost Optimization: Ensuring cost efficiency in a multi-cloud environment.
Governance, Security, and Compliance (GRC): Aligning policies across platforms.

#### Performance Considerations
Balancing performance needs between the cloud environments.
Step-by-Step Solution
Month 1-2: Assessment Phase

Objective: Understand the current state of both platforms and identify commonalities and differences.

Data Inventory: Create a detailed inventory of data assets, including tables, schemas, ETL processes, and metadata.
Performance and Cost Analysis: Analyze performance metrics and costs associated with each platform.
SQL Dialect Comparison: Identify similarities and differences in the SQL dialects used by Azure Synapse and Snowflake.
Month 3-4: Business Criticality Assessment

Objective: Prioritize data assets based on business impact and technical feasibility.

Critical Business Processes: Identify critical business processes that rely heavily on unified data access.
Stakeholder Interviews: Conduct interviews with key stakeholders to understand their requirements.
Technical Feasibility Analysis: Assess the technical complexity of unifying specific data assets.
Month 5-8: Unification of Key Data Assets

Objective: Integrate key data assets that are critical for business operations.

Data Integration Strategy: Develop a strategy for integrating common data assets, such as customer or product data.
ETL and Data Movement: Implement ETL processes to move data from Snowflake-on-AWS to Azure Synapse.
Performance Tuning: Optimize query performance on both platforms.
Month 9-12: Governance and Security Integration

Objective: Align governance, security, and compliance policies across the unified architecture.

Policies and Procedures: Develop or adapt existing policies for data governance, access control, and auditing.
Role-Based Access Control (RBAC): Implement RBAC to ensure proper user permissions.
Monitoring and Auditing: Set up monitoring tools to track usage patterns and compliance.
Month 13-16: Cost Optimization and Performance Scaling

Objective: Optimize costs while scaling performance for the unified architecture.

Cost Analysis Tools: Use cost analysis tools provided by both platforms to identify optimization opportunities.
Performance Tuning: Further optimize query performance, possibly through schema changes or indexing strategies.
Auto-scaling and Cost Management: Implement auto-scaling policies and budget alerts.
Month 17-20: Expansion and Additional Integration

Objective: Expand the unified architecture to include additional data assets and processes.

Additional Data Assets: Identify and integrate other critical data assets, such as financial or operational data.
Process Automation: Automate ETL pipelines and reporting processes to ensure consistency.
Continuous Improvement: Establish a continuous improvement cycle for ongoing optimization.
Month 21-24: Review and Handover

Objective: Finalize the unified architecture and hand it over to the operations team.

Documentation: Prepare comprehensive documentation, including data models, ETL processes, and governance policies.
Training Sessions: Conduct training sessions for stakeholders and users on the new system.
Review and Feedback: Gather feedback from key stakeholders and make final adjustments.

#### Closing Summary

By following this 24-month convergence strategy, we ensure a structured approach to unifying two independent data platforms while prioritizing critical business processes and technical feasibility. The plan covers assessment, integration, governance, cost optimization, and continuous improvement phases.

#### Common Mistakes/Follow-Ups
Ignoring Cost Optimization: Failing to address cost implications can lead to financial strain.
Over-Prioritizing Low-Impact Assets: Allocating too much effort on less critical data assets may divert focus from more impactful initiatives.
Lack of Continuous Improvement: Without setting up a cycle for ongoing optimization, the system may fall behind in terms of performance and compliance.

**Comparison:**

| Month | Phase               | Key Activities                                                                 |
|-------|---------------------|--------------------------------------------------------------------------------|
| 1-2   | Assessment          | Data inventory, performance analysis, SQL dialect comparison                    |
| 3-4   | Business Criticality| Identify critical processes, stakeholder interviews, technical feasibility      |
| 5-8   | Unification         | Integrate key data assets, ETL implementation                                   |
| 9-12  | Governance          | Develop policies, RBAC implementation, monitoring setup                         |
| 13-16| Cost Optimization   | Cost analysis, performance tuning                                               |
| 17-20| Expansion           | Additional data integration, process automation                                |
| 21-24| Handover            | Documentation, training sessions, feedback collection                          |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Month 1-2: Assessment Phase] --> B(Identify data inventory)
    B --> C[Azure Synapse Data Inventory]
    B --> D[Snowflake-on-AWS Data Inventory]
    C --> E[Performance Analysis Azure Synapse]
    D --> F[Performance Analysis Snowflake-on-AWS]
    E --> G[SQL Dialect Comparison]
    F --> H[SQL Dialect Comparison]

    A --> I[Month 3-4: Business Criticality Assessment]
    I --> J(Critical Business Processes)
    I --> K(Stakeholder Interviews)
    I --> L(Technical Feasibility Analysis)

    B --> M[Month 5-8: Unification of Key Data Assets]
    M --> N[Integrate Customer Data]
    M --> O[Integrate Product Data]

    C --> P[Month 9-12: Governance and Security Integration]
    P --> Q[Policies and Procedures]
    P --> R(Role-Based Access Control)
    P --> S[Monitoring and Auditing]

    A --> T[Month 13-16: Cost Optimization and Performance Scaling]
    T --> U(Cost Analysis Tools)
    T --> V(Performance Tuning)

    B --> W[Month 17-20: Expansion and Additional Integration]
    W --> X[Integrate Financial Data]
    W --> Y[Automate ETL Pipelines]

    A --> Z[Month 21-24: Review and Handover]
    Z --> AA(Documentation)
    Z --> AB(Training Sessions)
    Z --> AC(Review and Feedback)
```

**Example input / output:**

```text
Input:
- Azure Synapse data inventory
- Snowflake-on-AWS data inventory

Output:
- Integrated customer data from both platforms
- Automated ETL pipelines for financial reporting
- Comprehensive governance and security policies in place
```

**Data:**

```json
{
  "data_inventory": {
    "Azure_Synapse": [
      {"table_name": "customer_data", "columns": ["id", "name", "address"]},
      {"table_name": "product_data", "columns": ["id", "name", "price"]}
    ],
    "Snowflake_on_AWS": [
      {"table_name": "customer_data", "columns": ["id", "name", "address"]},
      {"table_name": "financial_data", "columns": ["year", "revenue"]}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Assessment Phase
6. Is Data Critical?
7. Integrate Key Assets
8. Is Process Feasible?
9. Integrate Data
10. Review and Feedback
11. Governance Integration
12. Cost Optimization
13. Performance Scaling
14. Handover Phase

