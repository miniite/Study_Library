---
type: "practice"
difficulty: "Basic"
---

# What is a labeled metric (with tags like host and region), and how would you filter a list of metric dicts in Python to only those from a given region?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Data Basics](<../../../../TOOLS/Python/Python_Language/Time-series_Data_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A labeled metric, often seen in monitoring systems, includes additional metadata such as host and region, which provide context to the metric value. In this problem, we are tasked with filtering a list of dictionaries representing these metrics based on a specific region.

#### Triaging the Question

The question is asking us to implement a function that filters a given list of dictionaries (each dictionary represents a metric) so that only those from a specified region remain. The core concepts involved are:

Understanding how to work with lists and dictionaries in Python.
Implementing filtering logic based on specific key-value pairs.

#### First Principles Solution

We need to iterate over the list of metrics, check if each dictionary has the required region tag, and include it in a new filtered list if it matches. Here’s the step-by-step solution:

Understand the Input: We have a list of dictionaries, where each dictionary contains keys like value, host, region, etc.
Define Filtering Logic: Create a function that takes two parameters: the list of metrics and the target region.
Implement the Filter Function: Use a list comprehension or a loop to filter the metrics based on the region.
Full Python Solution

Here's how we can implement this in Python:

PYTHON
def filter_metrics_by_region(metrics, target_region):
    """
    Filters a list of metric dictionaries by region.

    :param metrics: List of metric dictionaries (each dict has 'value', 'host', and 'region' keys)
    :param target_region: The region to filter the metrics by
    :return: A new list containing only the metrics that match the target region
    """
    return [metric for metric in metrics if metric['region'] == target_region]

# Example usage:
metrics = [
    {'value': 10, 'host': 'host1', 'region': 'us-west-2'},
    {'value': 20, 'host': 'host2', 'region': 'eu-central-1'},
    {'value': 30, 'host': 'host3', 'region': 'us-west-2'}
]

target_region = 'us-west-2'
filtered_metrics = filter_metrics_by_region(metrics, target_region)
print(filtered_metrics)  # Output: [{'value': 10, 'host': 'host1', 'region': 'us-west-2'}, {'value': 30, 'host': 'host3', 'region': 'us-west-2'}]

#### Closing Summary
and Common Mistakes

The solution is straightforward but requires attention to detail. Here are some common mistakes to avoid:

Forgetting to check the data structure of each metric dictionary.
Not properly handling edge cases, such as an empty input list or no matching regions.

#### Common Mistakes/Follow-Ups
Edge Cases: What happens if there are no metrics in the list?

#### Performance Considerations
If the list is very large, consider the performance of your filtering function.
Handling Missing Data: Ensure that the region key exists for each dictionary.

**Code:**

```python
def filter_metrics_by_region(metrics, target_region):
    return [metric for metric in metrics if metric['region'] == target_region]
```

**Example input / output:**

```text
metrics = [
    {'value': 10, 'host': 'host1', 'region': 'us-west-2'},
    {'value': 20, 'host': 'host2', 'region': 'eu-central-1'},
    {'value': 30, 'host': 'host3', 'region': 'us-west-2'}
]

target_region = 'us-west-2'
filtered_metrics = filter_metrics_by_region(metrics, target_region)
print(filtered_metrics)  # Output: [{'value': 10, 'host': 'host1', 'region': 'us-west-2'}, {'value': 30, 'host': 'host3', 'region': 'us-west-2'}]
```

**Data:**

```json
[
    {"value": 10, "host": "host1", "region": "us-west-2"},
    {"value": 20, "host": "host2", "region": "eu-central-1"},
    {"value": 30, "host": "host3", "region": "us-west-2"}
]
```

