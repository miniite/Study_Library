---
type: "practice"
difficulty: "Deep"
---

# Describe how you build a database vendor scorecard that stays current as the market and your workloads evolve over multiple years.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Database Vendor and Platform Choices](<../../../../01_Databases_and_SQL/Database_Strategy/Vendor_and_Tooling/Database_Vendor_and_Platform_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a robust database vendor scorecard, we need to ensure it's dynamic enough to adapt to evolving market conditions and workload needs. This involves assessing various dimensions of the database solutions and periodically updating them based on real-world performance metrics.

The core idea is to create a comprehensive framework that evaluates databases not just at one point in time but continuously as new features emerge and workloads change. We'll break down this process into key steps, starting from defining evaluation criteria to implementing automated updates.

#### Triage
What the Question Is Asking

This question probes how you would systematically assess database vendors over an extended period, considering multiple factors like performance, cost, scalability, support, and innovation. The goal is to create a vendor scorecard that remains relevant even as technology and business needs shift.

Step-by-Step Solution from First Principles

Define Evaluation Criteria:

Performance (TPS, latency, query optimization)
Cost (licensing models, maintenance, cloud pricing)
Scalability (horizontal vs vertical scaling options)
Support (SLA, customer service, community support)
Security and Compliance
Innovation (new features, open-source contributions)

Select Key Performance Indicators (KPIs):

For performance: Run benchmark tests using standard workloads.
For cost: Track licensing fees over a year.
For scalability: Measure how well the database scales with increasing data and load.

Gather Data from Multiple Sources:

Use public benchmarks like TPC-C, TPC-H, or industry reports.
Conduct internal tests using representative workloads.
Leverage real customer feedback through surveys and case studies.

Create a Scorecard Framework:

Develop a weighted score for each KPI based on importance.
Use a normalized scoring system (0-100) to compare vendors consistently.

Automate Data Collection and Analysis:

Set up monitoring tools that automatically collect data from test environments.
Implement scripts to run benchmarks at regular intervals.
Use dashboards to visualize trends over time.

Periodic Reviews and Updates:

Schedule quarterly or annual reviews of the scorecard.
Incorporate new features and technologies as they emerge.
Adjust weights based on changing business needs.

Document Decisions and Reasoning:

Maintain detailed logs for each update to ensure traceability.
Document rationale behind changes in scores and criteria.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By continuously updating the scorecard, we can make informed decisions about database vendors that align with our evolving business requirements. The key is maintaining a flexible yet rigorous approach that balances technical accuracy with practical applicability.

#### Common mistakes
include:

Focusing too narrowly on one or two criteria at the expense of others.
Not incorporating real-world usage data, leading to theoretical rather than practical assessments.
Ignoring emerging technologies and trends, causing the scorecard to become outdated quickly.

#### Follow-ups
might explore specific tools for automation, best practices in benchmarking, and how to integrate customer feedback effectively.

**Comparison:**

```
Define Evaluation Criteria
Select KPIs
Gather Data from MultipleSources
Create Scorecard Framework
Automate Data Collection andAnalysis
Periodic Reviews and Updates
Document Decisions andReasoning
```

**Code:**

```python
# Example Python script for running performance benchmarks
import psycopg2

def run_benchmark(db_name, user, password):
    conn = psycopg2.connect(
        dbname=db_name,
        user=user,
        password=password,
        host="localhost"
    )
    cursor = conn.cursor()

    # Run a representative query
    cursor.execute("SELECT COUNT(*) FROM customers")

    results = cursor.fetchall()
    print(results)
    cursor.close()
    conn.close()
```

**Example input / output:**

| Vendor | Performance (TPS) | Cost ($) | Scalability | Support (SLA) | Security |
|--------|------------------|---------|-------------|---------------|----------|
| PostgreSQL | 350 TPS | $1,200/yr | N/A | 9.5 / 10 | 8.7 / 10 |
| MySQL     | 400 TPS | $800/yr   | N/A       | 8.8 / 10 | 8.3 / 10 |
| Oracle    | 200 TPS | $2,500/yr | Horizontal | 9.0 / 10 | 9.5 / 10 |

**Data:**

```json
{
  "database_vendors": [
    {"name": "PostgreSQL", "score": 85},
    {"name": "MySQL", "score": 75},
    {"name": "Oracle", "score": 90}
  ],
  "kpi_weights": {
    "performance": 30,
    "cost": 25,
    "scalability": 15,
    "support": 15,
    "security": 10
  }
}
```

**Flow:**

1. +-------------------+
2. | Define Criteria   |
3. +-------+-----------+
4. |
5. v
6. +-------------------+
7. | Select KPIs       |
8. +-------+-----------+
9. |
10. v
11. +-------------------+
12. | Gather Data       |
13. +-------+-----------+
14. |
15. v
16. +-------------------+
17. | Create Framework  |
18. +-------+-----------+
19. |
20. v
21. +-------------------+
22. | Automate Collection|
23. +-------+-----------+
24. |
25. v
26. +-------------------+
27. | Periodic Updates  |
28. +-------+-----------+
29. |
30. v
31. +-------------------+
32. | Document Decisions|
33. +-------------------+

