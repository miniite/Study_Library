---
type: "practice"
difficulty: "Deep"
---

# How would you detect and quantify data drift or unexpected volume changes in a daily Hadoop feed before it corrupts downstream reports?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hadoop and Batch Processing](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/Hadoop_and_Batch_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the integrity of downstream reports, detecting and quantifying data drift or unexpected volume changes in a daily Hadoop feed is crucial. This involves monitoring the data schema, content, and volume to catch any discrepancies early. The process can be broken down into several steps: setting up a baseline, defining thresholds for anomalies, and implementing real-time or near-real-time monitoring.

The key idea here is to establish a robust system that can proactively identify issues before they propagate downstream. This not only helps in maintaining data quality but also ensures that critical business decisions are based on reliable information.

**Comparison:**

```
Check Type	Thresholds
Volume Change	±120% of baseline volume
Schema Changes	±5 fields differ
```

**Example input / output:**

```text
# Running the check_data_drift function with sample data
check_result = check_data_drift(current_feed, baseline_feed)
print(check_result)  # Output: (True, {'volume': 0.25, 'schema': ['id']})
```

**Flow:**

1. Yes
2. Anomaly Detected
3. No Anomaly
4. Start
5. Is data feed available?
6. Initialize baseline
7. Monitor daily Hadoop feed
8. Calculate volume difference
9. Check schema integrity
10. Detect anomalies based onthresholds
11. Alert and notify stakeholders
12. Update baseline with currentfeed

