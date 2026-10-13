---
type: "practice"
difficulty: "Basic"
---

# Why might you run unit tests on a metrics-parsing script in CI before it goes to a server that processes telemetry?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pre-merge vs Post-merge Failures](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pre-merge_vs_Post-merge_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure robust and reliable software, we often incorporate unit tests into our Continuous Integration (CI) pipeline for scripts that process metrics or telemetry data. In this case, running unit tests on a metrics-parsing script in CI before it goes to the server is crucial because:

Early Detection of Bugs: Unit tests help catch bugs early, reducing the likelihood of issues being introduced upstream.
Code Quality and Reliability: Ensuring that the parsing logic works correctly prevents incorrect data from reaching production systems, which could lead to faulty insights or decisions based on inaccurate telemetry.

Let's break down why this is important step-by-step:

**Code:**

```python
def parse_metrics(line):
    # Example function to parse metrics
    try:
        metric_name, value = line.split(":")
        return {"name": metric_name.strip(), "value": float(value)}
    except ValueError:
        raise ValueError("Invalid metric format")

# Example unit test for the parse_metrics function
import unittest

class TestMetricsParsing(unittest.TestCase):
    def test_valid_input(self):
        self.assertEqual(parse_metrics("cpu:85.0"), {"name": "cpu", "value": 85.0})

    def test_invalid_input(self):
        with self.assertRaises(ValueError):
            parse_metrics("network")
```

**Example input / output:**

```text
Input:
- Test input line: "cpu:85.0"
- Expected output: {"name": "cpu", "value": 85.0}

Output:
- Actual result: {"name": "cpu", "value": 85.0}
```

**Data:**

```json
[
    {
        "line": "cpu:85.0",
        "expected_result": {"name": "cpu", "value": 85.0}
    },
    {
        "line": "network",
        AIOps Fresher|AIOps 2/3
    }
]
```

