---
type: "practice"
difficulty: "Deep"
---

# Your warehouse spend is dominated by a handful of exploratory notebooks doing full-table cross joins. As principal, what architectural guardrails (query cost limits, sampling defaults, required partition filters, result caching) do you put in without killing analyst productivity, and how do you measure the trade-off?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Small Files and Compaction](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a Database Developer Architect/Principal, when dealing with high-cost exploratory queries that dominate our warehouse spend, we need to focus on optimizing both cost efficiency and user productivity without compromising on essential analysis capabilities. The key is to implement strategic guardrails that guide query behavior while ensuring analysts can still perform necessary operations.

The main goal here is to balance between reducing unnecessary costs (like full-table cross joins) and maintaining the flexibility and speed required for data exploration. We need to introduce mechanisms such as query cost limits, sampling defaults, partition filters, and result caching. These guardrails should be carefully crafted to minimize disruption while ensuring that analysts can still derive value from their queries.

Firstly, query cost limits are essential. By setting a reasonable threshold on the maximum allowable execution time or resource usage for exploratory queries, we discourage full-table scans and cross joins unless absolutely necessary. This limit acts as an immediate safeguard against runaway expensive queries.

Secondly, sampling defaults can help reduce the dataset size without significantly impacting the analysis quality. By default, queries should only retrieve a portion of the data (e.g., 10-20% of rows), which is sufficient for quick exploration and hypothesis testing. This approach ensures that analysts can still run their experiments but in a more cost-effective manner.

Thirdly, partition filters are crucial. Given that our warehouse likely contains large datasets partitioned by time or other relevant dimensions, we should enforce the use of these partitions when making queries. For instance, if an analyst is exploring data from a specific month or year, their query should automatically include appropriate partition filters to reduce the amount of data processed.

Lastly, result caching can significantly improve performance and cost efficiency. When analysts run similar queries multiple times, caching results can save significant compute resources. However, we must ensure that this does not lead to stale data by implementing proper invalidation strategies based on changes in underlying datasets or metadata.

To measure the trade-off, we need to monitor both query performance metrics and financial costs. We can use a combination of tools like AWR (Automatic Workload Repository) reports, SQL trace analysis, and cost monitoring services provided by cloud providers. Key metrics include:

Query Execution Time: Identifies slow queries that might benefit from optimization.
Resource Consumption: Tracks CPU, memory, and I/O usage to identify high-cost operations.
Cost Analysis: Measures the monetary impact of expensive queries on the organization.

We should also conduct periodic reviews with data analysts to understand their workflows and gather feedback on how these guardrails are affecting their productivity. This iterative process will help us fine-tune our settings over time, ensuring that we strike the right balance between cost efficiency and user satisfaction.

**Comparison:**

```
Partition Key	Partitions Count	Data Size (GB)
Date	365	47.8
Time	24	10.2
User ID	1000	3.5
```

**Code:**

```sql
-- Example of setting a query cost limit in SQL
ALTER SYSTEM SET QUERY_COST_LIMIT = 10000;
```

**Example input / output:**

| Query | Execution Time (s) | Resource Usage (%) |
|-------|--------------------|-------------------|
| Full-Table Cross Join A, B | 3600 | 75% CPU, 90% I/O |
| Sampled Cross Join A, B | 120 | 15% CPU, 20% I/O |

**Flow:**

1. Yes
2. No
3. User runs query
4. Is the query expensive?
5. Notify DBA for optimization
6. Query executes with guardrailsapplied
7. Result cached

