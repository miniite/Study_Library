---
type: "practice"
difficulty: "Expert"
---

# How do you balance building a central observability lake versus per-team lakes and what governance problems does centralization solve or create?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Mesh vs Central Observability Lake](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're balancing the pros and cons of a centralized observability data lake against having per-team lakes. A centralized approach can provide unified visibility, standardization, and efficiency, but it also introduces complexities in governance, security, and operational overhead. Conversely, per-team lakes offer flexibility and autonomy but can lead to data silos, inconsistencies, and difficulty in maintaining a holistic view.

The key is understanding the trade-offs and choosing the architecture that best aligns with your organization’s goals, scale, and cultural norms. Centralization solves governance problems by ensuring consistency across teams and reducing the risk of shadow IT. However, it also creates challenges such as complex data lineage and potential bottlenecks in data access.

**Comparison:**

```
Metrics	Centralized Lake	Per-Team Lakes
Consistency	High	Low
Scalability	Moderate	High
Governance	Strong	Weak
Flexibility	Low	High
```

**Code:**

```python
# Example code snippet to illustrate a simple ingestion pipeline for a centralized lake
def ingest_data(streams):
    # Process each stream and store in the central lake
    for stream in streams:
        process_stream(stream)
        save_to_lake(stream)

# Example function to handle individual team data
def process_stream(stream):
    # Basic processing logic
    cleaned_data = clean_data(stream)
    return cleaned_data

def save_to_lake(data):
    # Save the processed data into a central lake
    with open('central_lake.csv', 'a') as file:
        file.write(','.join(data) + '\n')
```

**Example input / output:**

```text
Input:
Stream of telemetry data from a DevOps team

Output:
Processed data written to the central observability lake, with proper metadata and lineage tracking.
```

**Data:**

```json
{
  "data_lake": {
    "type": "central",
    "teams": [
      { "name": "DevOps", "lake_name": "devops_lake" },
      { "name": "Security", "lake_name": "security_lake" }
    ],
    "governance": {
      "policies": ["access_control", "data_retention"],
      "tools": ["IAM policies", "audit logs"]
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Centralized Lake
8. Is Data Standardized?
9. Unified View and Analysis
10. Custom Processing Required
11. Per-Team Lakes
12. Do Teams Require DifferentViews?
13. Data Silos, InconsistentStandards
14. Autonomy and Customization
15. Standardize Data
16. Audit Logs for Compliance
17. Governance Policies Applied
18. Per-Team Lakes
19. Teams Need IndependentViews?
20. Silos, Shadow IT Risks
21. Local Processing and Flexibility

