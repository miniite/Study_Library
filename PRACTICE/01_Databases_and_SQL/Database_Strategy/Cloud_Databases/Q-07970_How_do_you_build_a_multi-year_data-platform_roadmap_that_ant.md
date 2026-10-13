---
type: "practice"
difficulty: "Expert"
---

# How do you build a multi-year data-platform roadmap that anticipates Aurora and Cosmos feature evolution without betting the company on unreleased or preview features?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cloud Database Architecture](<../../../../01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Cloud_Database_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When building a multi-year data-platform roadmap, especially in the context of cloud databases like Amazon RDS, Amazon Aurora, and Microsoft Azure Cosmos DB, it's crucial to balance forward-looking innovation with robust risk management. The key is to identify strategic features that are likely to become mainstream while avoiding premature commitment to preview or unreleased functionalities. This involves a layered approach: understanding current capabilities, projecting future trends, and iteratively refining the roadmap based on feedback and evolving requirements.

#### Triage

The question asks how to develop a long-term strategy for cloud databases, specifically focusing on Amazon Aurora and Azure Cosmos DB, without relying solely on experimental features. The interviewer is probing your ability to:

Predict Future Features: Identify which upcoming features are likely to be adopted by the mainstream.
Risk Management: Develop strategies to mitigate risks associated with unreleased or preview features.
Technical Depth: Demonstrate a deep understanding of both Amazon Aurora and Azure Cosmos DB, as well as broader cloud database trends.

#### Assumptions
The current state: Both Amazon Aurora and Azure Cosmos DB are mature cloud databases with robust feature sets.
Future projections: These databases will continue to evolve but at varying speeds and with different priorities.
Company needs: The company has specific data storage, performance, and scalability requirements.

#### First-Principles Solution
1. Current State Analysis
Amazon Aurora: Highly optimized for MySQL and PostgreSQL workloads; benefits from AWS’s DevOps expertise.
Azure Cosmos DB: NoSQL document database with automatic global distribution, multi-model support (SQL, MongoDB, Cassandra, Gremlin).
Both are widely used but have distinct strengths.
2. Identifying Key Features
Amazon Aurora:
Performance Enhancements: Optimizations like storage and query performance.
Security Improvements: Enhanced encryption and authentication mechanisms.
Cost Reduction: Cost-efficient deployment options and auto-scaling.
Azure Cosmos DB:
NoSQL Support: Expansion of supported models (Cassandra, Gremlin).
Global Databases: Improved global distribution capabilities.
Performance Tiers: Different performance tiers for cost optimization.
3. Future Trends
Amazon is likely to focus on database management and AI integrations.
Azure might continue enhancing NoSQL support and expanding its multi-model offerings.
4. Risk Mitigation Strategies
Pilot Projects: Implement preview features in non-critical environments first.
Regular Reviews: Periodically reassess roadmap based on feature releases and company needs.
Vendor Partnerships: Collaborate with vendors to get early access and feedback on upcoming features.
5. Iterative Roadmap Development
Define a flexible roadmap that can be adjusted as needed.
Prioritize features based on business value, technical feasibility, and risk.
Plan for both short-term gains and long-term strategic moves.

#### Closing Summary

A comprehensive multi-year data-platform roadmap should leverage the strengths of Amazon Aurora and Azure Cosmos DB while managing risks associated with unreleased or preview features. This involves a balanced approach that includes current state analysis, identifying key future trends, implementing pilot projects, regular reviews, and flexible planning.

#### Common Mistakes/Follow-Ups
Mistake: Over-reliance on unreleased features without proper validation.

#### Follow-up
Discuss the importance of pilot projects and regular review cycles to validate new features before full-scale adoption.
Mistake: Ignoring vendor partnerships for early access and feedback.

#### Follow-up
Emphasize how vendor relationships can provide valuable insights into upcoming releases.

**Comparison:**

| Feature | Aurora | Cosmos DB |
|---------|--------|----------|
| Performance Enhancements | High | Medium |
| Security Improvements | High | Low |
| Cost Reduction | High | Low |
| NoSQL Support | Low | High |
| Global Databases | Medium | High |

**Code:**

```python
# Example code snippet for a pilot project setup
import boto3

def create_pilot_instance():
    rds = boto3.client('rds')
    response = rds.create_db_instance(
        DBInstanceIdentifier='pilot-instance',
        Engine='aurora-postgresql',
        MasterUsername='admin',
        MasterUserPassword='password123'
    )
    return response

response = create_pilot_instance()
print(response)
```

**Example input / output:**

```text
Input:
create_pilot_instance()

Output:
{
  "DBInstance": {
    "DBInstanceIdentifier": "pilot-instance",
    "Engine": "aurora-postgresql",
    "MasterUsername": "admin",
    "Status": "available"
  }
}
```

**Data:**

```json
{
  "DBInstance": {
    "DBInstanceIdentifier": "pilot-instance",
    "Engine": "aurora-postgresql",
    "MasterUsername": "admin",
    "Status": "available"
  }
}
```

**Flow:**

1. Current State Analysis
2. Pilot Projects
3. Regular Reviews
4. Vendor Partnerships
5. Iterative RoadmapDevelopment
6. Adaptation and Feedback

