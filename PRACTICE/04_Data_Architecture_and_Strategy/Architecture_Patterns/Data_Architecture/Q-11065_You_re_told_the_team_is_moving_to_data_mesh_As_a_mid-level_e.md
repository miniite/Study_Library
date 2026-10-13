---
type: "practice"
difficulty: "Intermediate"
---

# You're told the team is 'moving to data mesh.' As a mid-level engineer whose team now owns its data as a product, what actually changes about your day-to-day responsibilities?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-domain Joins in a Mesh](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Cross-domain_Joins_in_a_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In moving from traditional data warehousing to a data mesh architecture, several key aspects of your daily responsibilities will shift significantly. At a high level, the primary change is that you'll now own and manage data as a product, making sure it's accessible, usable, and aligned with business needs across different teams.

#### Triaging the Question

The team’s transition to "data mesh" implies a fundamental transformation in how data is managed, accessed, and utilized. As a mid-level engineer, your role will involve more than just technical implementation; you’ll be tasked with aligning data strategies with business goals. The key areas of change include ownership, governance, access control, and data usability.

Core Idea

The core idea behind the data mesh is to decentralize data management while centralizing metadata and service-level agreements (SLAs). This means that each domain within your organization will become an independent data owner, responsible for its own data governance and quality.

#### First-Principles Solution

Ownership and Governance: Historically, data might have been siloed in a few large warehouses managed by central teams. Now, you'll be part of a domain-specific team that owns the data from ingestion to consumption. This shift means you must ensure data integrity, lineage, and security within your domain.

Data Cataloging and Metadata: In a data mesh, metadata becomes crucial for discovery and usage. You’ll need to implement robust cataloging systems (e.g., using tools like AWS Glue or Azure Data Catalog) that capture detailed information about each dataset’s source, schema, lineage, and ownership. This will enable cross-domain teams to understand the data they need.

Service-Level Agreements: Domain-specific teams must define clear SLAs with other domains. These agreements will cover things like response times for data requests, availability guarantees, and quality metrics (e.g., accuracy, completeness). For example, a marketing team might require real-time access to customer behavior data, while finance requires historical financial data.

APIs and Data Services: Instead of centralizing all data processing, you'll create APIs that expose your domain's data services. These APIs should be well-documented and secured to ensure only authorized users can access the necessary data. For instance, a product team might need an API for real-time inventory updates.

Data Quality and Validation: You’ll focus on ensuring high-quality data within your domain. This includes setting up pipelines for data validation, monitoring, and remediation. Tools like Apache Nifi or Prefect can be used to automate these processes.

Collaboration and Communication: Working in a data mesh often means collaborating more closely with other domains and business units. You’ll need to build strong relationships and establish effective communication channels (e.g., regular meetings, dashboards) to ensure everyone is aligned on data usage and quality.

#### Closing Summary

In summary, the move to a data mesh requires you to adopt a more holistic approach to data management that includes domain-specific ownership, metadata governance, service-level agreements, secure APIs, and high-quality data practices. This shift will also involve significant changes in how teams collaborate and communicate about their data needs and usage.

#### Common Mistakes/Follow-Ups

Overlooking Domain-Specific Needs: One common mistake is treating all domains equally when implementing the mesh architecture. Each domain has unique requirements that must be carefully considered.

Ignoring Security: Secure APIs and strict access controls are essential in a data mesh but can sometimes be overlooked due to their complexity.

Lack of Clear SLAs: Without clear service-level agreements, it’s easy for different teams to have conflicting expectations about data availability and quality.

Insufficient Metadata Management: Poor metadata management can lead to data silos and make it difficult for other domains to understand or use your data effectively.

Poor Collaboration: Effective communication is crucial, but often, there can be a lack of cross-domain collaboration leading to misaligned expectations about data usage and governance.

#### Example
Workflow

To illustrate, consider an example workflow in the context of an e-commerce company:

PLAINTEXT
1. Domain: Marketing
   - Data Ingestion: Real-time customer behavior logs from various touchpoints.
   - Data Processing: ETL pipelines to transform raw logs into actionable insights (e.g., user journey analysis).
   - Data Services: RESTful API for real-time marketing campaigns based on user segments.

2. Domain: Finance
   - Data Ingestion: Historical financial transaction data from multiple systems.
   - Data Processing: Aggregation and cleaning of transactional data to provide accurate financial reports.
   - Data Services: SQL endpoints for ad-hoc queries by finance analysts.

3. Collaboration:
   - SLA Agreement: Marketing needs real-time customer behavior updates every 5 minutes, while Finance requires nightly batch processing for accurate reporting.

**Comparison:**

```
Domain	Data Ingestion	Data Processing	Data Services
Marketing	Real-time customer logs	User journey analysis	RESTful API for real-time marketing insights
Finance	Historical transaction data	Aggregated and cleaned financial reports	SQL endpoints for ad-hoc queries
```

**Code:**

```python
# Example ETL Pipeline in Apache Nifi
from nifi import NiFi

def process_customer_data():
    ni = NiFi()
    # Ingest raw logs from various sources
    source = ni.get_source("customer_logs")
    cleaned = ni.clean(source)
    insights = ni.process(cleaned, "journey_analysis")
    api = ni.create_api(insights, endpoint="/marketing/api/v1/segments")
```

**Example input / output:**

```text
Input:
- Customer behavior logs from multiple touchpoints

Output:
- Real-time API for marketing campaigns
- Aggregated SQL endpoints for financial reporting
```

**Data:**

```json
{
  "metadata": {
    "source": "customer_behavior_logs",
    "schema": [
      {"name": "user_id", "type": "string"},
      {"name": "event_time", "type": "timestamp"},
      {"name": "action", "type": "string"}
    ],
    "lineage": ["marketing_touchpoint_1.csv", "marketing_touchpoint_2.json"],
    "ownership": "Marketing Team"
  },
  "slas": {
    "response_time": "5 minutes",
    "availability": "99.9%",
    "quality_metrics": [
      {"metric": "accuracy", "target": 0.98},
      {"metric": "completeness", "target": 0.95}
    ]
  }
}
```

**Flow:**

1. Data Ingestion
2. Filter Data
3. Clean Data
4. Transform Data
5. Generate Insights
6. Provide API
7. Store in Warehouse
8. Process Financial Logs
9. Aggregate and Clean Data
10. Generate Reports
11. SQL Endpoints for Ad-HocQueries

