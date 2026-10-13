---
type: "practice"
difficulty: "Deep"
---

# A cost-review finds two teams independently running near-identical data-enrichment pipelines. How do you architect and govern the consolidation into a shared service without either team losing autonomy or blaming FinOps for the reorg?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Cost Governance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Scaling_Cost_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to consolidate two near-identical data-enrichment pipelines while ensuring that both teams retain their autonomy and the FinOps department doesn't face blame from either team. The key is to design a shared service architecture that respects the current workflows of both teams and aligns with best practices in cloud cost governance.

The primary goal is to leverage a microservices-based approach, where each team's pipeline can run independently but share common components like data sources, storage, or processing logic. This way, we maintain flexibility and autonomy for both teams while reducing redundancy and optimizing costs.

#### Triage
Understand the Problem: Both teams have near-identical pipelines, indicating potential redundancies.
Identify Key Stakeholders: Teams responsible for data enrichment, FinOps, DevOps, and possibly compliance or governance teams.
Define Objectives:

#### Cons
olidate pipelines to reduce costs.
Ensure both teams retain autonomy in their workflows.
Maintain alignment with cloud cost governance principles.

#### First-Principles Solution
Identify Common Components: Determine which parts of the data-enrichment pipeline can be shared, such as data sources, storage layers, and processing logic.
Design Microservices: Create a microservice architecture where each team’s unique steps remain in their pipelines, but common components are abstracted into reusable services.
Implement Shared Storage: Use managed cloud storage solutions to ensure both teams can access the same datasets without duplicating storage infrastructure.
Centralized Monitoring and Governance: Set up centralized monitoring tools to track usage and costs associated with shared resources, ensuring transparency and accountability.
Detailed Steps

Identify Common Components:

Data sources: Both teams might be accessing similar data from external APIs or databases.
Storage layers: They likely use the same types of storage solutions like S3 or managed object storage.
Processing logic: There may be common steps in enriching and preparing data.

Design Microservices:

Create a service for each shared component, e.g., DataFetcherService for pulling data from sources, PreprocessorService for initial processing, etc.
Ensure these services are designed to be stateless and scalable.

Implement Shared Storage:

Use cloud-based object storage solutions like AWS S3 or Azure Blob Storage to store datasets.
Grant appropriate permissions to both teams so they can access the same data without duplicating storage infrastructure.

Centralized Monitoring and Governance:

Implement a centralized monitoring system (e.g., CloudWatch, Prometheus) to track usage of shared resources.
Set up alerts for excessive or unexpected costs associated with shared services.
Define policies for cost optimization and resource utilization, ensuring both teams adhere to these guidelines.

#### Closing Summary

By leveraging microservices architecture and centralized governance tools, we can effectively consolidate the data-enrichment pipelines while respecting the autonomy of each team. This approach not only reduces redundancy but also ensures transparency and accountability, aligning with best practices in cloud cost governance.

#### Common Mistakes/Follow-Ups
Over-Engineering: Avoid overcomplicating the shared services to the point where they become inflexible or hard to maintain.
Lack of Clear Policies: Ensure that clear policies are defined and communicated regarding usage, costs, and compliance.
Resistance from Teams: Address concerns early on by involving both teams in the design process to ensure buy-in.

**Comparison:**

```
Service	Purpose	Teams Using	Shared Components
DataFetcherService	Fetch and prepare data	Both Teams	Storage
PreprocessorService	Process enriched data	Both Teams	None
```

**Code:**

```python
# Example configuration for a microservice to fetch data
class DataFetcherService:
    def __init__(self, config):
        self.config = config

    def fetch_data(self):
        # Implementation to fetch data from an external source
        pass

# Example configuration for a preprocessor service
class PreprocessorService:
    def __init__(self, storage_client):
        self.storage_client = storage_client

    def preprocess_data(self, raw_data):
        # Implementation to process the fetched data and store it in the shared storage
        return processed_data
```

**Example input / output:**

```text
Input:
- Team A requests data from the `DataFetcherService` and processes it with the `PreprocessorService`.
- Team B uses a different entry point for the same services.

Output:
- Both teams share access to the same storage layer, reducing redundancy.
- Centralized monitoring shows cost optimization across both teams.
```

**Data:**

```json
{
  "storage": {
    "type": "S3",
    "bucket": "shared-data-bucket"
  },
  "services": [
    {
      "name": "DataFetcherService",
      "dependencies": ["config"]
    },
    {
      "name": "PreprocessorService",
      "dependencies": ["storage_client"]
    }
  ]
}
```

**Flow:**

1. Team A
2. DataFetcherService
3. PreprocessorService
4. Shared S3 Bucket
5. Team B
6. DataFetcherService
7. PreprocessorService
8. Shared S3 Bucket

