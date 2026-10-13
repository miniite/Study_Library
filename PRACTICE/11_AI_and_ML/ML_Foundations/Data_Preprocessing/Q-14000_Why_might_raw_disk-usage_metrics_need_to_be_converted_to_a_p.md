---
type: "practice"
difficulty: "Basic"
---

# Why might raw disk-usage metrics need to be converted to a percentage of total capacity before comparing across hosts?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Preprocessing Fundamentals](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Preprocessing_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why converting raw disk-usage metrics to a percentage of total capacity is necessary before comparing them across different hosts, we must first grasp how disk usage can be misleading when not normalized. Raw metrics like bytes used or gigabytes (GB) are insufficient for comparison because they do not account for differences in the total available capacity on each host's disk. This oversight could lead to incorrect conclusions and ineffective resource management.

#### Triaging the Question

The question is probing your understanding of data preprocessing and normalization, specifically why it's important to convert raw metrics into a relative measure before analysis. The assumptions here are that you're dealing with disk usage across multiple hosts, each potentially having different total capacities. We need to normalize these values to ensure they can be meaningfully compared.

#### First-Principles Solution

Understanding Raw Metrics: Disk usage in bytes or GB does not provide a complete picture of how much capacity is being utilized. For example, Host A with 20GB used out of 50GB total might seem less critical than Host B with the same amount (20GB), but only on the assumption that Host B has 1TB total.

Normalization to Percentages: Converting these raw metrics to a percentage of the total capacity provides a standardized view. For instance, converting the above examples:

Host A: ( \frac{20}{50} \times 100 = 40% )
Host B: ( \frac{20}{1TB} \approx 2%)

Comparison Across Hosts: Once normalized, we can compare the percentage usage effectively. In our example, even though both hosts use the same absolute amount of space, Host A is more critically overusing its available space.

#### Closing Summary

Converting raw disk-usage metrics to a percentage of total capacity ensures that comparisons are meaningful and not skewed by differences in the host's disk size. This normalization allows us to identify which hosts are experiencing significant resource strain relative to their available capacity, enabling better decision-making for resource management.

#### Common Mistakes/Follow-Ups
Forgetting Total Capacity: Not accounting for the total disk capacity can lead to incorrect assessments of storage health.
Ignoring Edge Cases: Pay attention to edge cases such as hosts with very small or large capacities that might disproportionately affect overall storage efficiency if not normalized.

#### Follow-Up
Questions: The interviewer may ask about other normalization techniques, data cleaning steps, or how this approach impacts real-world monitoring and alerting systems.

**Comparison:**

```
Host	Raw Usage (GB)	Total Capacity (GB)	Normalized Usage (%)
A	20	50	40.0
B	20	1000	2.0
```

**Code:**

```python
# Example Python code to convert raw disk usage to percentage of total capacity

def normalize_disk_usage(raw_bytes_used, total_capacity):
    return (raw_bytes_used / total_capacity) * 100

host_A_raw_usage = 20_000_000  # 20GB
host_A_total_capacity = 50_000_000  # 50GB
host_B_raw_usage = 20_000_000  # 20GB
host_B_total_capacity = 1_000_000_000_000  # 1TB

# Normalizing the usage
host_A_normalized_usage = normalize_disk_usage(host_A_raw_usage, host_A_total_capacity)
host_B_normalized_usage = normalize_disk_usage(host_B_raw_usage, host_B_total_capacity)

print(f"Host A: {host_A_normalized_usage:.2f}%")
print(f"Host B: {host_B_normalized_usage:.2f}%")
```

**Example input / output:**

```text
Input:
- HostA with 20GB used out of 50GB total capacity.
- HostB with 20GB used out of 1TB (1000GB) total capacity.

Output:
- Host A: 40.00%
- Host B: 2.00%

Explanation:
- Raw metrics suggest both hosts are using the same absolute amount, but normalized percentages show significant differences in relative usage.
```

**Data:**

```json
{
  "hosts": [
    {
      "hostname": "HostA",
      "raw_bytes_used": 20_000_000,
      "total_capacity_gb": 50
    },
    {
      "hostname": "HostB",
      "raw_bytes_used": 20_000_000,
      "total_capacity_gb": 1000
    }
  ]
}
```

**Flow:**

1. Yes
2. Raw Disk Usage
3. Normalize?
4. Calculate as Percentage ofTotal Capacity
5. Compare Across Hosts?
6. Identify Resource Strain
7. Ignore Differences in TotalCapacity
8. Incorrect Assessments

