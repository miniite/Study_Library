---
type: "practice"
difficulty: "Deep"
---

# How do you handle high-cardinality tags on custom metrics that are inflating cost while still keeping useful dimensions?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ APM and Trace Analytics](<../../../../TOOLS/Datadog/Datadog_Platform/APM_and_Trace_Analytics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with high-cardinality tags in custom metrics using Datadog, we need to balance between maintaining useful metadata and controlling costs. The primary challenge here is understanding how these tags are inflating your cost without compromising on the insights they provide. This involves a mix of strategic tagging practices, data aggregation techniques, and possibly implementing some form of rate limiting or dimension sampling.

To approach this problem from first principles:

Understanding the Problem: High-cardinality tags refer to those that have many unique values, such as user IDs or session IDs. These can rapidly increase the number of time series Datadog tracks, leading to increased storage costs and potentially performance issues.

#### Assumptions
You are using Datadog for monitoring and alerting on custom metrics.
The high-cardinality tags are essential for business insights but need to be managed to avoid excessive cost.

We will explore a few key steps to manage this:

Tag Filtering: Implement tag filtering rules in Datadog to exclude certain low-value tags from the full metric set. This can help reduce the number of time series being monitored.
Dimension Sampling: Use dimension sampling to limit the amount of data collected for high-cardinality tags, ensuring you still capture valuable insights while reducing storage costs.
Aggregation Strategies: Employ aggregation strategies such as using sum, count, or custom aggregations based on the nature of your metrics and business needs.

Let's dive deeper into each approach:

Tag Filtering:

You can configure Datadog to automatically filter out certain tags by adding them to a list of excluded dimensions.

#### Example
If you have a metric for user activity, exclude specific users or sessions that are not critical for your monitoring.

Dimension Sampling:

Datadog supports sampling strategies like count and distribution, which can help reduce the data volume while preserving useful insights.
For instance, if a tag represents a session ID, you might sample only 10% of these sessions to get a representative view.

Aggregation Strategies:

Use aggregation functions that make sense for your metrics. For example, count could be appropriate for metrics where you're interested in the total number of events.
You can also use custom scripts or Lambda functions to process data before sending it to Datadog, applying further filtering and aggregations.

#### Closing Summary

In summary, handling high-cardinality tags effectively requires a combination of strategic tagging practices and data management techniques. By using tag filtering, dimension sampling, and appropriate aggregation strategies, you can maintain useful dimensions while keeping costs in check.

#### Common Mistakes/Follow-Ups
Mistake: Over-reliance on raw metric collection without considering the implications.

#### Follow-Up
Discussing how to determine the right balance between cost and insight based on business needs. This might involve A/B testing different configurations or using Datadog's alerting rules to monitor cost trends.

**Comparison:**

```
Tag	Description
user_id	Unique identifier for each user session.
device_type	Type of device used by the user (e.g., mobile, desktop).
```

**Code:**

```python
# Example Python script for sampling tags before sending metrics to Datadog
import random

def sample_data(metric, tags):
    if 'user_id' in tags:
        # Sample 10% of user IDs
        if random.random() < 0.1:
            return metric, {key: value for key, value in tags.items() if key != 'user_id'}
    return metric, tags

# Usage example
metric = "user_activity"
tags = {'user_id': '12345', 'device_type': 'mobile'}
filtered_metric, filtered_tags = sample_data(metric, tags)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `{"metric": "user_activity", "tags": {"user_id": "12345", "device_type": "mobile"}}` | `{ "metric": "user_activity", "tags": { "device_type": "mobile" } }` |

**Data:**

```json
{
  "metric": "user_activity",
  "tags": {
    "user_id": "12345",
    "device_type": "mobile"
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is high-cardinality tag critical?
5. Apply tag filtering andsampling
6. Keep full tagging
7. Monitor cost and adjust asneeded
8. Monitor and optimize asneeded

