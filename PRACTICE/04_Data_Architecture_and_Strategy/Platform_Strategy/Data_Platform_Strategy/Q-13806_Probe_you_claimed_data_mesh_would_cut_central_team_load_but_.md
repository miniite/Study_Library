---
type: "practice"
difficulty: "Expert"
---

# Probe: you claimed data mesh would cut central team load — but now every domain reinvents ingestion. How do you keep 'federated' from meaning 'duplicated 40 times'? What must stay centralized?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Platform-level Data Quality](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Platform-level_Data_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Data Mesh architecture, the goal is to decentralize data management while maintaining governance and control at a central level. The crux of your question revolves around ensuring that federated systems do not become duplicated efforts, especially when every domain starts managing its own ingestion processes. To address this challenge, we need to strike a balance between decentralization and centralization.

Firstly, let's clarify what the interviewer is probing: they are interested in understanding how you can maintain a unified data strategy across multiple domains while preventing each one from creating independent systems that could lead to duplication of effort and inefficiency. The key here is to identify which aspects should remain centralized to ensure consistency, governance, and cost-effectiveness.

**Comparison:**

```
Centralized Components	Decentralized Components
Metadata Management	Ingestion Processes
Data Quality	ETL Workflows
Lineage Tracking	Data Storage and Processing
```

**Example input / output:**

```text
# Example Input -> Output for Centralized Components
Input: User request to implement data quality checks across multiple domains.
Output: Unified data quality rules and policies defined centrally, with domain-specific implementations.

Example Request: "Central team implements a new data validation rule."
Response: "Unified rule applied globally, with localized adjustments for specific use cases."
```

**Data:**

```yaml
# Sample Data Model for Centralized vs Decentralized Components
centralized_components:
  - Metadata Management
  - Data Quality
  - Lineage Tracking

decentralized_components:
  - Ingestion Processes
  - ETL Workflows
  - Data Storage and Processing
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD
3. A[Central Team] -- Manages -> B(Metadata)
4. B -- Ensures -> C(Data Quality)
5. C -- Tracks -> D(Lineage)
6. A -- Supports --> E[Domain Teams]
7. E -- Uses --> F(Ingestion)
8. E -- Processes --> G(ETL)
9. E -- Stores --> H(Data Storage)

