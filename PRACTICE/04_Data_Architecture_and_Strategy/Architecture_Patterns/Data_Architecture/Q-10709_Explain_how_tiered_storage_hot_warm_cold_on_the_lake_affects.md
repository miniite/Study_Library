---
type: "practice"
difficulty: "Intermediate"
---

# Explain how tiered storage (hot/warm/cold) on the lake affects your ability to run a training job over old data, and what surprises you on cost or latency.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Lake Storage](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In modern data architectures, especially those involving large-scale datasets like in machine learning (ML), tiered storage plays a crucial role. By separating your data into "hot," "warm," and "cold" tiers based on access frequency, you can optimize cost and performance for training jobs over old data. This strategy involves storing frequently accessed data closer to the application, while less frequently accessed data is stored further away.

#### Triaging the Question

The question aims to understand how tiered storage strategies affect ML training jobs. Specifically, it explores two key dimensions: cost (e.g., storage and computational costs) and latency (e.g., time taken for data retrieval). The interviewer might be probing your understanding of:

Data access patterns: How often different types of data are accessed.
Cost implications: Trade-offs between storage tiers and the associated costs.
Latency considerations: Impact on training job performance due to different storage tiers.

#### Assumptions
The dataset is large, with varying access frequencies for different parts of the data.
Storage tiers are defined as "hot," "warm," and "cold."
Hot: Frequently accessed data.
Warm: Less frequently accessed but still needed occasionally.
Cold: Rarely accessed historical data.
Tiered Storage in Data Architecture

Hot Tier: Typically, the most expensive storage tier with high I/O performance. Examples include NVMe SSDs or RAM-based storage. This is where you keep your current and frequently used models for training and inference.

Warm Tier: A less expensive tier than the hot tier but still optimized for faster access times compared to cold tiers. It might use slower SSDs or a combination of faster and cheaper media, such as HDDs with RAID configurations.

Cold Tier: The least expensive storage tier, often used for long-term archiving or infrequently accessed data. This could be tape storage, cloud object storage, or even more cost-effective cloud services like AWS S3 Glacier.

#### First-Principles Solution

To run a training job over old data, you need to consider the following:

Data Access Patterns:

For hot and warm tiers: You can frequently access recent data without significant latency issues.
For cold tiers: Accessing infrequently used historical data will incur higher latencies due to slower storage speeds.

Cost Implications:

Training on the hot tier is optimal for real-time or near-real-time applications but expensive.
Using the warm tier can reduce costs while maintaining acceptable access times, making it suitable for periodic training jobs.
Cold tiers are the most cost-effective option for historical data, which can be accessed less frequently.

Latency Considerations:

Hot and warm tiers provide low latency but come with higher costs.
Cold tiers offer lower costs at the expense of increased access times, making them suitable for batch processing or periodic training jobs.
Practical Implementation

To implement tiered storage in an ML lake:

Data Classification: Classify data based on access patterns using metadata tagging or automated rules.
Storage Strategy: Store hot and warm data closer to the compute resources (e.g., on-premises servers, local SSDs).
Cost Optimization: Use cold tiers for long-term archiving and infrequently accessed historical data.

#### Example Scenario

Suppose you have a dataset of 1PB, with recent models requiring frequent access while older models are rarely used:

Hot Tier (200GB): Models and datasets frequently accessed during real-time training.
Warm Tier (300GB): Historical models that need occasional access but not as often as the hot tier.
Cold Tier (500GB+): Infrequently accessed historical data stored off-site.
Cost Analysis

Assuming storage costs:

Hot Tier: $1/GB/month
Warm Tier: $0.10/GB/month
Cold Tier: $0.01/GB/month

For a 1PB dataset, the cost breakdown would be:

Hot Tier: $200/month
Warm Tier: $300/month
Cold Tier: $5,000/month (for the cold storage)
Performance Impact
Latency: Accessing hot and warm tiers will have low latency, suitable for real-time or near-real-time training.
Training Time: Cold tier access might delay training jobs by a few minutes due to slower I/O speeds.

#### Closing Summary

Tiered storage is an effective strategy in ML data architectures. By balancing cost and performance through the use of hot, warm, and cold tiers, you can optimize your operations for both frequent and infrequent data access needs. However, it's important to regularly review and adjust your tiering strategy based on actual access patterns and business requirements.

#### Common Mistakes
Ignoring Access Patterns: Failing to classify data properly leads to suboptimal storage choices.
Over-Engineering Cold Tiers: Using overly expensive cold tiers for data that could be accessed more frequently.
Neglecting Performance Trade-offs: Overly prioritizing cost savings without considering the impact on training job performance.

#### Follow-Ups
How do you ensure the accuracy of access patterns?
What are your recommendations for automating tiering decisions in an ML environment?
How would you handle data that crosses tiers frequently?

**Comparison:**

```
Tier	Size (GB)	Cost per GB/month	Monthly Cost
Hot	200	$1	$200
Warm	300	$0.10	$30
Cold	580,000	$0.01	$5,800
```

**Code:**

```python
# Example Python code to simulate hot/warm/cold tier usage
def get_storage_cost(size_gb, tier):
    if tier == "hot":
        return size_gb * 1.00
    elif tier == "warm":
        return size_gb * 0.10
    else:
        return size_gb * 0.01

# Simulate a 1PB dataset
total_size = 1_000_000  # 1 PB in GB
hot_tier_size = 200
warm_tier_size = 300
cold_tier_size = total_size - (hot_tier_size + warm_tier_size)

# Calculate costs
cost_hot = get_storage_cost(hot_tier_size, "hot")
cost_warm = get_storage_cost(warm_tier_size, "warm")
cost_cold = get_storage_cost(cold_tier_size, "cold")

total_cost = cost_hot + cost_warm + cost_cold
print(f"Total Cost: ${total_cost:.2f}")
```

**Example input / output:**

```text
Input:
- Dataset size: 1PB (1,000,000 GB)
- Hot tier: 200GB
- Warm tier: 300GB
- Cold tier: 580,000GB

Output:
- Cost for hot tier: $200/month
- Cost for warm tier: $300/month
- Cost for cold tier: $5,800/month
- Total cost: $6,300/month
```

**Data:**

```json
{
  "dataset": {
    "hot_tier": "200GB",
    "warm_tier": "300GB",
    "cold_tier": "500GB"
  },
  "storage_costs": {
    "hot_tier": "$1/GB/month",
    "warm_tier": "$0.10/GB/month",
    "cold_tier": "$0.01/GB/month"
  }
}
```

**Flow:**

1. Data Access Pattern
2. Identify Hot Data
3. Classify into Tiers
4. Save to Hot Tier
5. Save to Warm Tier
6. Save to Cold Tier
7. Monitor and Adjust

