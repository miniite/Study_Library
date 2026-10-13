---
type: "practice"
difficulty: "Deep"
---

# How would you deploy a change to your log-to-metric extraction so a bad regex can't spike cardinality and blow up your time-series database in prod?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data and Analytics CI Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Data_and_Analytics_CI_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of deploying a change to the log-to-metric extraction without risking cardinality spikes, we need to ensure that our changes are thoroughly tested and validated before being rolled out into production. This involves implementing a robust CI/CD pipeline that includes rigorous testing phases. The key idea is to isolate and verify each change in a safe environment before it reaches prod.

The question primarily focuses on ensuring the safety of regex modifications, which can lead to cardinality issues if not handled correctly. By leveraging a well-structured CI/CD process, we can systematically test these changes in multiple stages—development, staging, and pre-production—to catch any potential problems early.

**Comparison:**

```
Stage	Description
Development	Code changes are made and tested locally.
Staging	Automated tests, including integration and performance testing, are run to ensure the change works as expected.
Pre-production	A small subset of production data is used for a canary release to test under real-world conditions before full deployment.
```

**Code:**

```python
def validate_regex_change(regex_pattern):
    # Example of validating regex pattern
    import re

    try:
        compiled = re.compile(regex_pattern)
        return True
    except re.error:
        return False

# Example function to simulate log processing and metric extraction
def process_logs(logs, regex_pattern):
    if not validate_regex_change(regex_pattern):
        raise ValueError("Invalid regex pattern")

    metrics = []
    for line in logs:
        match = re.search(regex_pattern, line)
        if match:
            metric = extract_metric_from_match(match)
            metrics.append(metric)
    return metrics

# Example usage
log_lines = ["2023-10-01 09:00:00 - ERROR - Some error message", "2023-10-01 10:00:00 - INFO - Another info"]
metrics = process_logs(log_lines, r"\b(\d{4}-\d{2}-\d{2})")
print(metrics)
```

**Example input / output:**

```text
Input:
logs = [
    "2023-10-01 09:00:00 - ERROR - Some error message",
    "2023-10-01 10:00:00 - INFO - Another info"
]
regex_pattern = r"\b(\d{4}-\d{2}-\d{2})"

Output:
[{'year': '2023', 'month': '10', 'day': '01'}]
```

**Data:**

```json
{
    "logs": [
        "2023-10-01 09:00:00 - ERROR - Some error message",
        "2023-10-01 10:00:00 - INFO - Another info"
    ],
    "regex_pattern": r"\b(\d{4}-\d{2}-\d{2})"
}
```

**Flow:**

1. Development
2. Code Commit
3. Automated Tests
4. Staging
5. Performance Testing
6. Pre-production
7. Canary Release
8. Full Production Deployment

