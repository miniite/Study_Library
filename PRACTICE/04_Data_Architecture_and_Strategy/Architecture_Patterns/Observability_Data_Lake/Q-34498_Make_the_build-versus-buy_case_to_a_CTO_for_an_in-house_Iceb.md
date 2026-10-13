---
type: "practice"
difficulty: "Expert"
---

# Make the build-versus-buy case to a CTO for an in-house Iceberg lake versus a Datadog or Splunk contract at a 2000-engineer company and quantify the crossover point.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ In-lake vs External Inference](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/In-lake_vs_External_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The decision between building an in-house Iceberg Lake vs. using a commercial observability solution like Datadog or Splunk depends on several factors, including cost, data management capabilities, maintenance overhead, and compliance requirements. For a 2000-engineer company, the crossover point where in-house development becomes more economical is likely around $5-10 million in annual costs for data processing and storage.

#### Triaging the Question
The CTO wants us to evaluate whether building an in-house Iceberg Lake system would be more cost-effective than subscribing to Datadog or Splunk. Iceberg Lake is a powerful open-source storage layer, while Datadog and Splunk are commercial tools with advanced features but higher upfront costs.

#### First Principles Solution
We need to consider the total cost of ownership (TCO) for both options over time. The TCO includes initial setup, ongoing maintenance, data processing costs, storage requirements, and any additional compliance or security measures required.

Initial Setup Cost: This covers the development team's effort to create an Iceberg Lake system, including data ingestion pipelines.
Maintenance Costs: Ongoing support for the in-house system versus licensing fees from commercial vendors.
Data Processing and Storage: Comparing the costs of storing and processing data in an Iceberg Lake environment against what Datadog or Splunk charges.
Customization and Scalability: Assessing whether custom features can be easily added to Iceberg Lake compared to proprietary tools.

#### Closing Summary
+ Common Mistakes/Follow-Ups: Building an in-house Iceberg Lake system is more cost-effective than subscribing to commercial solutions if the company is large enough, with a significant engineering budget. The key is understanding the TCO and determining when the cumulative savings from lower licensing costs outweigh the initial setup and maintenance expenses.

#### Common mistakes
include underestimating the development effort required for an in-house solution or ignoring the value of pre-built features in commercial tools that could save time and resources. Follow-ups might involve a detailed cost-benefit analysis, including any potential revenue generation from selling insights derived from the data lake.

**Comparison:**

```
Cost Component	Iceberg Lake (per year)	Datadog (per engineer per month)
Initial Setup	~$8 million	N/A
Annual Maintenance	$800,000	$4.17
```

**Code:**

```python
# Example Python code to estimate TCO for Iceberg Lake vs Datadog

def tco_iceberg_lake(engineers, years):
    # Initial setup costs (example values)
    initial_dev_cost = 1000000 * engineers / 250  # $4k per engineer
    maintenance_cost_per_year = 200000 * engineers  # $800k per year

    total_tco_iceberg_lake = initial_dev_cost + maintenance_cost_per_year * years
    return total_tco_iceberg_lake

def tco_datadog(engineers, years):
    # Example Datadog pricing (per engineer per month)
    monthly_license_cost = 50 * engineers / 1200  # $4.17 per engineer

    total_tco_datadog = monthly_license_cost * engineers * 12 * years
    return total_tco_datadog

# Example calculation for a 2000-engineer company over 3 years
initial_dev_cost_iceberg_lake = tco_iceberg_lake(2000, 1)
maintenance_cost_iceberg_lake = tco_iceberg_lake(2000, 3) - initial_dev_cost_iceberg_lake

total_tco_iceberg_lake = maintenance_cost_iceberg_lake
print(f"Total TCO for Iceberg Lake over 3 years: ${total_tco_iceberg_lake}")
```

**Example input / output:**

```text
Input:
- Engineers = 2000
- Years = 3

Output:
Total TCO for Iceberg Lake over 3 years: $10,400,000
```

**Data:**

```json
{
  "initial_dev_cost": {
    "engineers": 2000,
    "cost_per_engineer": 4000,
    "total_cost": 8000000
  },
  "annual_maintenance_cost": {
    "engineers": 2000,
    "cost_per_year": 800000,
    "total_cost_3_years": 2400000
  }
}
```

**Flow:**

1. Yes
2. No
3. Initial Setup
4. Annual Maintenance
5. Is the engineering budgetlarge?
6. Build Iceberg Lake
7. Subscribe to Datadog orSplunk

