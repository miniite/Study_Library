---
type: "practice"
difficulty: "Expert"
---

# You're asked to cut database spend 30% in two quarters without an SLA regression. How do you sequence the levers, and which do you deliberately avoid because they trade cost for risk?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Database Cost Optimization](<../../../../01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of cutting database spend by 30% over two quarters while maintaining service level agreements (SLAs), we need to carefully sequence our optimizations and prioritize those that do not compromise SLAs or introduce unacceptable risks. The key levers include query optimization, resource management, storage tiering, and potentially some form of data reduction. However, certain strategies should be avoided if they risk degrading performance.

**Comparison:**

```
Lever	Description	Risk
Query Optimization	Analyzing and rewriting inefficient queries.	Low, if done carefully.
Resource Management	Adjusting CPU and memory allocations.	Medium, potential for service interruptions.
Storage Tiering	Migrating data to cheaper storage tiers.	High, risk of increased latency.
Data Reduction	Reducing the volume of stored data.	Very high, risk of SLA degradation.
```

**Code:**

```python
# Example: Query Optimization
def optimize_queries(database):
    # Analyze slow queries using EXPLAIN
    slow_queries = analyze_slow_queries(database)
    for query in slow_queries:
        optimized_query = rewrite_query(query)
        update_database(query, optimized_query)

# Example: Resource Management
def manage_resources(database):
    # Set appropriate resource limits and configurations
    set_resource_limits(database, cpu_limit=4096, memory_limit=8192)
```

**Example input / output:**

```

Example input and output for resource management:

Input:

PYTHON
Copy
database = get_database_instance()
manage_resources(database)

Output:

PLAINTEXT
Copy
Database resources updated with new limits.
```

**Data:**

```yaml
- slow_queries:
  - query_id: "123"
    execution_time: "10s"
  - query_id: "456"
    execution_time: "15s"
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Query Optimization
6. Analyze Slow Queries
7. Are Queries Optimized?
8. Done
9. Rewrite and Deploy OptimizedQueries
10. Resource Management
11. Set Resource Limits
12. Are Resources Sufficient?
13. Done
14. Adjust Resource AllocationStrategy
15. Monitor Performance
16. Review and Iterate

