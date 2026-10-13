---
type: "practice"
difficulty: "Deep"
---

# What is the difference between design-time and runtime lineage, and when has each mattered in an incident you handled?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lineage for Dynamic SQL](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Lineage_for_Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design-time and runtime lineage are fundamental concepts in data governance that help trace the origin and flow of data throughout its lifecycle. Design-time lineage focuses on understanding how data is transformed from source to target, including steps like ETL jobs or data pipelines. Runtime lineage, on the other hand, captures real-time data flows as they happen, which can be crucial for troubleshooting issues in a production environment.

In an incident I handled, both types of lineage were instrumental. We had a critical bug that caused inaccurate financial reports to be generated. The issue seemed like it was happening during the ETL process but wasn't immediately clear where the problem originated. By leveraging design-time lineage, we mapped out all the ETL jobs and transformations involved in generating these reports.

Next, I used runtime lineage to trace data flows through the system as they happened. This allowed us to pinpoint which exact piece of transformed data was causing the issue by comparing it against known good states from previous runs. By combining insights from both types of lineage, we were able to identify a subtle bug in our data cleaning script that wasn't properly handling null values.

**Code:**

```python
# Example pseudo-code for runtime lineage tracking
class DataFlow:
    def __init__(self, source, target):
        self.source = source
        self.target = target

    def log_event(self, event_data):
        # Log the transformation applied and result
        pass

# Example of tracing a data flow in real-time
flow = DataFlow(source='raw_sales_data', target='processed_sales_report')
flow.log_event(event_data={'timestamp': '2023-10-05T14:00:00Z', 'transformation': 'null_value_removal'})
```

**Example input / output:**

```text
Input:
- Raw sales data with null values and inconsistent date formats.

Output:
- Final sales report showing accurate financial figures, but with discrepancies noted in certain regions.
```

**Data:**

```json
{
  "data_flow": [
    {
      "source": "raw_sales_data",
      "target": "preprocessed_sales_data",
      "transformations": ["remove_nulls", "convert_date_format"]
    },
    {
      "source": "preprocessed_sales_data",
      "target": "final_sales_report",
      "transformations": ["aggregate_by_region", "apply_commission_rules"]
    }
  ]
}
```

