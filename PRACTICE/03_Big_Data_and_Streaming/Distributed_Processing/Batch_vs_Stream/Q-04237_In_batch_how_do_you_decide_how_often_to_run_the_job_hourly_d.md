---
type: "practice"
difficulty: "Intermediate"
---

# In batch, how do you decide how often to run the job (hourly, daily, weekly)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Batch vs Streaming Cost Trade-offs](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding on the frequency of running a batch processing job, the key is to balance the trade-offs between data freshness, resource utilization, and operational complexity. The core idea is to choose an interval that ensures your insights are current enough while keeping the overhead manageable.

The question essentially probes how you understand the implications of different frequencies (e.g., hourly vs daily) on both the business needs and technical capabilities. Let's break this down step-by-step:

Understand Business Requirements: The primary goal is to provide timely insights that drive decision-making. For instance, if your business requires real-time or near-real-time analytics, you might need more frequent runs.
Data Volume Considerations: More frequent runs can increase resource consumption and complexity in managing those resources (e.g., ETL processes).
Resource Utilization: Balancing between using resources effectively and avoiding over-provisioning is crucial. For example, if your data volume doubles weekly but remains relatively constant daily, a daily job might be sufficient.

Let's consider an example:

#### Example Scenario
You are working on a retail company that needs to process sales data for inventory management.
Business Need: Timeliness of the insights is critical as outdated information can lead to stockouts or overstocking.
Data Volume: The daily volume of new transactions is around 10,000 records.
Step-by-Step Reasoning:

Determine Data Freshness Requirements:

If timely inventory decisions are crucial and the business needs data no older than a day, hourly jobs might be overkill.
Weekly updates could be too late for fast-moving retail environments.

Estimate Resource Impact:

Hourly processing would require running 24 jobs per day, which increases the load on resources but ensures very fresh insights.
Daily processing reduces the overhead to one job per day, making it more resource-efficient while still providing reasonably fresh data (12 hours old).

Evaluate Failure Modes and Costs:

Hourly processing can lead to higher operational costs due to increased ETL load.
Daily processing might miss some immediate insights but is less costly in terms of resources.

Operational Complexity: More frequent jobs require better monitoring, error handling, and potentially more robust infrastructure setup (e.g., scheduling, logging).

#### Conclusion

For this retail example, a daily job might be the best compromise between timeliness and resource efficiency. If you find that even daily data is too late for critical decisions, you may need to reevaluate your system architecture or consider integrating streaming processing for real-time insights.

#### Common Mistakes

Choosing an interval without understanding the business needs.
Overprovisioning resources due to frequent runs.
Ignoring the complexity of managing more frequent processes.

**Code:**

```python
# Example Python script to simulate job scheduling (for illustration)
def schedule_job(frequency):
    if frequency == 'hourly':
        print("Running hourly jobs")
    elif frequency == 'daily':
        print("Running daily jobs")

schedule_job('daily')
```

**Flow:**

1. Define Business Needs
2. Determine Data Freshness
3. Estimate Resource Impact
4. Evaluate Failure Modes andCosts
5. Schedule Jobs Based onTrade-offs

