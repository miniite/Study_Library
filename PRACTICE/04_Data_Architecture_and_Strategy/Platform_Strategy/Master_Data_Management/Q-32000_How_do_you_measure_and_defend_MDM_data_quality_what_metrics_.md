---
type: "practice"
difficulty: "Deep"
---

# How do you measure and defend MDM data quality — what metrics (match rate, false-merge rate, completeness, uniqueness, stewardship backlog) do you put on an exec dashboard and what SLAs do you commit?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ MDM for Batch and Streaming](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/MDM_for_Batch_and_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure and defend Master Data Management (MDM) data quality, we need to establish a robust set of metrics that align with business objectives and provide actionable insights. These metrics should be presented on an executive dashboard to ensure transparency and accountability. Let's break down the key metrics, their definitions, and how they contribute to MDM governance.

#### Triaging the Question

The question is asking about the specific metrics used in MDM to measure data quality and the Service Level Agreements (SLAs) that should be established for these metrics. The interviewer is probing your understanding of best practices in MDM, your ability to articulate key performance indicators (KPIs), and your approach to setting realistic and meaningful SLAs.

#### First-Principles Solution
Match Rate: This metric measures the accuracy of data matching across multiple sources.
False-Merge Rate: Indicates the frequency of incorrect merges between similar records, which can lead to data redundancy and inconsistencies.
Completeness: Ensures that all necessary fields are filled out in the MDM system.
Uniqueness: Avoids duplicate entries by ensuring each record is unique based on a set of critical attributes.
Stewardship Backlog: Tracks the backlog of records requiring manual review and validation.
Detailed Metrics
1. Match Rate
Definition: The percentage of correctly matched records between source systems and the MDM system.
Calculation: (Correct Matches / Total Possible Matches) * 100%
SLA Example: Aim for at least 95% match rate to ensure high accuracy.
2. False-Merge Rate
Definition: The percentage of records incorrectly merged into the same record in the MDM system.
Calculation: (False Merges / Total Possible Merges) * 100%
SLA Example: Aim for a false-merge rate below 2% to minimize data redundancy.
3. Completeness
Definition: The percentage of records with all mandatory fields populated.
Calculation: (Complete Records / Total Records) * 100%
SLA Example: Aim for at least 98% completeness across critical fields.
4. Uniqueness
Definition: The percentage of unique records based on a set of critical attributes.
Calculation: (Unique Records / Total Records) * 100%
SLA Example: Aim for a uniqueness rate above 95%.
5. Stewardship Backlog
Definition: The number or percentage of records awaiting manual review and validation by data stewards.
Calculation: Number of Unreviewed Records / Total Number of Records * 100%
SLA Example: Aim to reduce the backlog to less than 5% of total records.
Executive Dashboard

These metrics should be visualized on an executive dashboard with clear, actionable KPIs. For example:

Metric	Target (SLA)	Current Value
Match Rate	≥95%	[Current %]
False-Merge Rate	≤2%	[Current %]
Completeness	≥98%	[Current %]
Uniqueness	≥95%	[Current %]
Stewardship Backlog	<5%	[Current %]

#### Closing Summary

By establishing and regularly monitoring these metrics, we can ensure that our MDM system is robust and reliable. The executive dashboard provides a clear picture of the data quality status and helps in making informed decisions to improve the overall health of the master data.

#### Common Mistakes/Follow-Ups
Ignoring Edge Cases: Failing to account for edge cases like partial matches or complex record structures.
Lack of SLA Enforcement: Not setting clear SLAs or not enforcing them, leading to poor performance.
Insufficient Data Stewardship: Overlooking the importance of having active and engaged data stewards.

**Comparison:**

    | Metric          | Target (%) | Current Value (%) |
    |-----------------|-----------|------------------|
    | Match Rate      | ≥95       | [current value]   |
    | False-Merge Rate| ≤2        | [current value]   |
    | Completeness    | ≥98       | [current value]   |
    | Uniqueness      | ≥95       | [current value]   |
    | Stewardship     | <5        | [current value]   |

**Code:**

```python
# Example code for calculating match rate
def calculate_match_rate(correct_matches, total_possible_matches):
    return (correct_matches / total_possible_matches) * 100

match_rate = calculate_match_rate(980, 1020)
print(f"Match Rate: {match_rate}%")
```

**Example input / output:**

```text
Input:
Match Rate: 94.8%
False-Merge Rate: 2.3%
Completeness: 97.5%
Uniqueness: 94.2%
Stewardship Backlog: 6.1%

Output:
Current MDM Data Quality Status:

- Match Rate: 94.8% (target: ≥95%)
- False-Merge Rate: 2.3% (target: ≤2%)
- Completeness: 97.5% (target: ≥98%)
- Uniqueness: 94.2% (target: ≥95%)
- Stewardship Backlog: 6.1% (target: <5%)
```

**Data:**

```json
{
  "metrics": [
    {
      "name": "Match Rate",
      "target": 95,
      "current_value": 94.8
    },
    {
      "name": "False-Merge Rate",
      "target": 2,
      "current_value": 2.3
    },
    {
      "name": "Completeness",
      "target": 98,
      "current_value": 97.5
    },
    {
      "name": "Uniqueness",
      "target": 95,
      "current_value": 94.2
    },
    {
      "name": "Stewardship Backlog",
      "target": 5,
      "current_value": 6.1
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. No
4. No
5. No
6. No
7. Start
8. Is match rate >= 95%?
9. Metric达标
10. Improve Matching Logic
11. Is false-merge rate <=2%?
12. Metric达标
13. Reduce False Merges
14. Are all critical fields complete?
15. Metric达标
16. Populate Missing Fields
17. Are records unique?
18. Metric达标
19. Implement Uniqueness Checks
20. Is stewardship backlog <5%?
21. Metric达标
22. Reduce Stewardship Backlog
23. End

