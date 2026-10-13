---
type: "practice"
difficulty: "Deep"
---

# What indexing or bloom-filter strategy would you apply to lake files so log-search for incident forensics doesn't scan terabytes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Telemetry Lake File Layout](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Telemetry_Lake_File_Layout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the issue of efficient log search in an Observability Data Lake, we need a strategy that balances between minimizing storage costs and ensuring quick access to logs during incident forensics. The key is to implement effective indexing or Bloom filter strategies without bloating the data lake.

This question requires us to understand the trade-offs between storage, query performance, and false positives in search. We will reason through this from first principles, discussing both indexing and Bloom filters before arriving at a practical solution.

**Comparison:**

| Time        | Path              | Message                                                                                                                                                      |
|-------------|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2023-10-01T15:00:00 | /data/operations/system_logs.csv | User failed to authenticate (error) at 15:00:00                                                                                                                |

**Code:**

```
MERMAID
Copy
graph TB
    A[Log Data Lake] --> B[Raw Logs]
    B --> C[Indexing Strategy]
    B --> D[Bloom Filter Strategy]
    C --> E[Reduced Scan Time]
    D --> F[Reduced False Positives]
    C -- "Balanced" --> G[Optimized Indexing]
    D -- "Tuned" --> H[Tuned Bloom Filters]
```

**Example input / output:**

```text
Given a set of log files in the data lake:

- **Input:** A request to search for logs with the keyword "error" within the last week.
- **Output:** The system should quickly return relevant logs without scanning terabytes of data.

| Time       | Path                        | Content                |
|------------|-----------------------------|------------------------|
| 2023-10-01 | /var/log/application.log    | {"timestamp": "2023-10-01T14:00:00", "level": "info", "message": "User logged in"} |
| 2023-10-01 | /data/operations/system_logs.csv | User failed to authenticate (error) at 15:00:00 |
| 2023-10-01 | /logs/security/access.log   | {"timestamp": "2023-10-01T16:00:00", "level": "warning", "message": "Invalid login attempt from 192.168.1.1"} |
```

**Data:**

```yaml
logLake:
  - path: /var/log/application.log
  - path: /data/operations/system_logs.csv
  - path: /logs/security/access.log
```

**Flow:**

1. True Positive
2. False Positive
3. Log Search Request
4. Timestamp Filter
5. Bloom Filter Match
6. Index Lookup
7. Retrieve Logs
8. Re-check Against Index
9. No Match, Return No Results

