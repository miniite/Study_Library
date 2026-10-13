---
type: "practice"
difficulty: "Intermediate"
---

# Tell me about a time your mesh design assumption was wrong at scale; what broke and what did you change in the architecture?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Mesh Principles](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Mesh_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In my role as a Data Engineer, I once faced an interesting challenge while designing a Data Mesh architecture for our company's data platform. The assumption that led to problems at scale was that decentralizing data management and governance would inherently solve the issues of data fragmentation and inconsistency across teams. However, we quickly realized that this approach had unforeseen consequences.

The core of the issue was the lack of proper coordination mechanisms between different teams. Each team had its own set of data sources, processing pipelines, and storage locations, leading to a proliferation of duplicate or conflicting data. The assumption was that self-service and autonomy would naturally lead to better outcomes without explicit governance structures.

To address this problem, we had to reevaluate our architecture and introduce several changes:

Unified Data Governance Framework: We implemented a centralized governance framework that provided guidelines on data standards, metadata management, and security policies.
Inter-Team Coordination Tools: We deployed tools for inter-team communication and collaboration, such as Slack channels and shared dashboards to ensure that teams could coordinate effectively.
Centralized Metadata Repository: We created a metadata repository that captured the lineage of data assets and their transformations across different pipelines, ensuring traceability.

These changes significantly improved the quality and consistency of our data across the organization.

**Comparison:**

| Data Asset    | Description | Lineage                         |
|---------------|-------------|--------------------------------|
| CustomerSales | Sales data  | CRM -> ETLProcess -> S3         |
| ProductPrices | Pricing info| ERP -> DataLake -> SFTP         |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Data Mesh Design] --> B[Decentralized Data Management];
    B --> C[Self-Service Teams];
    C --> D[Data Fragmentation];
    C --> E[Inconsistent Data];
    F[Centralized Governance Framework] --> G[Metadata Standards];
    G --> H[Data Traceability];
    I[Inter-Team Coordination Tools] --> J[Shared Dashboards];
    J --> K[Improved Communication];
    K --> L[Enhanced Collaboration];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Customer ID: 12345, Region: North America, Sales Date: 2023-09-15 | Sales record for customer 12345 in the North America region on September 15, 2023. |

**Data:**

```yaml
metadata_repository:
  data_assets:
    - name: CustomerSales
      description: Sales data for different regions.
      lineage:
        source_system: CRM
        processing_pipeline: ETLProcess
        storage_location: S3
        last_updated: "2023-10-01"
```

**Flow:**

1. Data Source
2. EtlPipeline
3. StorageLayer
4. MetadataRepo
5. GovernanceFramework
6. InterTeamTool
7. CommunicationChannel
8. CollaborationPlatform

