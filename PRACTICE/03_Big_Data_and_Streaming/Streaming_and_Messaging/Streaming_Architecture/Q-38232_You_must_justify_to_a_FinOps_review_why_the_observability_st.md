---
type: "practice"
difficulty: "Expert"
---

# You must justify to a FinOps review why the observability streaming platform consumes 12% of the cloud bill. Break the spend into its structural drivers and tell me which are irreducible physics versus which are architectural debt you'd pay down.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Streaming for AIOps](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Streaming_for_AIOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When justifying to a FinOps review why the observability streaming platform consumes 12% of the cloud bill, we need to break down the spend into its structural drivers and categorize them into irreducible physics (fixed costs) versus architectural debt that can be optimized. This analysis will help us understand where we are spending our money and how we can improve efficiency.

#### Triage
The key question here is identifying and justifying cloud cost components in a streaming architecture, specifically for an observability platform. The interviewer is probing your ability to decompose costs, assess their nature (fixed vs. variable), and provide rational arguments for each component. We will assume the system includes compute, storage, network, and other services typically involved in real-time data processing.

#### First-principles solution

Compute: Analyze the compute cost based on the number of instances running, their size, and utilization rates.
Storage: Break down the storage costs by types (e.g., cold vs. hot data) and usage patterns.
Network: Evaluate network costs related to data transfer between nodes or to external services.
Other Services: Consider any additional services like logging, monitoring, and management tools.
Compute Costs

Compute costs are driven by the number of instances running, their size, and utilization rates. For example:

Number of Instances: 50 (average)
Instance Size: t3.small with a compute utilization rate of 40%

For each t3.small instance costing $0.012 per hour, the total cost is approximately $6.24 per day.

Storage Costs

Storage costs depend on data volume and cold/hot storage ratios.

Data Volume: 50 TB (90% hot, 10% cold)
Hot Storage Cost: $0.0233 per GB-month; Cold Storage Cost: $0.0046 per GB-month

For the above volumes:

Hot Storage: ( 50 \times 0.9 \times 0.0233 \approx $11.28 ) monthly
Cold Storage: ( 50 \times 0.1 \times 0.0046 \approx $0.23 ) monthly

Total storage cost: ( $11.51 )

Network Costs

Network costs are driven by data transfer volume and rates.

Data Transfer: 1 TB/day at an average rate of $0.09 per GB
Total network cost: ( 1,024 \times 0.09 = $93 ) monthly
Other Services

Other services include logging, monitoring, and management tools:

Logging: $50/month
Monitoring: $75/month
Management Tools: $30/month

Total other service cost: ( $155 )

Summary of Breakdown
Compute: $249.60 per month (for 50 t3.small instances)
Storage: $11.51 per month
Network: $93 per month
Other Services: $155 per month

Total Monthly Cost: $609.61

Irreducible Physics vs. Architectural Debt

Irreducible Physics (Fixed Costs):

Storage Costs: These are directly related to the data volume and its lifecycle, which is a fundamental aspect of observability.
Compute Costs: The number of instances running is necessary for processing real-time data, making it an essential component.

Architectural Debt:

Network Optimization: Reducing unnecessary data transfer or optimizing network configurations could save costs without compromising functionality.
Instance Sizing and Utilization: Improving instance utilization rates can reduce the number of instances needed, thereby lowering compute costs. For example, using smaller instances with better CPU or GPU optimizations.

#### Closing Summary

The majority of the cloud bill for the observability streaming platform is due to storage and compute costs, which are essential components of real-time data processing. However, there is room for optimization in network usage and instance sizing, which could be addressed through architectural improvements. These changes would help reduce overall costs without impacting the core functionality of the observability system.

#### Common Mistakes/Follow-Ups
Overlooking Storage Class Costs: Failing to differentiate between hot and cold storage can lead to unnecessary expenses.
Neglecting Network Optimization: Ignoring data transfer optimizations might result in higher costs than necessary.
Focusing Only on Compute: Overlook other critical components like network and storage, which also contribute significantly to the overall cost.

**Comparison:**

| Component      | Cost Breakdown                         |
|----------------|----------------------------------------|
| Compute        | $249.60 per month                      |
| Storage (Hot)  | $11.51 per month                       |
| Storage (Cold) | $0.23 per month                        |
| Network        | $93 per month                          |
| Other Services | $155 per month                         |
| **Total**      | **$609.61 per month**                  |

**Code:**

```python
# Example Python script to calculate storage costs
data_volume = 50 * 1024  # in GB
hot_storage_percentage = 0.9
cold_storage_percentage = 0.1

storage_cost_hot = data_volume * hot_storage_percentage * 0.0233 / (30)  # monthly cost
storage_cost_cold = data_volume * cold_storage_percentage * 0.0046 / (30)  # monthly cost

total_storage_cost = storage_cost_hot + storage_cost_cold
```

**Example input / output:**

```text
# Example input and output for storage cost calculation
data_volume = 50 * 1024  # in GB
hot_storage_percentage = 0.9
cold_storage_percentage = 0.1

storage_cost_hot = data_volume * hot_storage_percentage * 0.0233 / (30)  # monthly cost
storage_cost_cold = data_volume * cold_storage_percentage * 0.0046 / (30)  # monthly cost

total_storage_cost = storage_cost_hot + storage_cost_cold

print(f"Total Storage Cost: ${total_storage_cost:.2f} per month")
```

**Data:**

```json
{
  "data_volume": 50,
  "hot_storage_percentage": 0.9,
  "cold_storage_percentage": 0.1,
  "hot_storage_cost_per_gb_month": 0.0233,
  "cold_storage_cost_per_gb_month": 0.0046
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Compute Cost] --> B1{Number of Instances};
4. B1 --> C[t3.small @ $0.012/hour, Utilization 40%];
5. C --> D[$6.24/day, $249.60/month];
6. E[Storage Cost] --> F1{Data Volume};
7. F1 --> G[50 TB (90% Hot, 10% Cold)];
8. G --> H[Hot Storage @ $0.0233/GB-month];
9. G --> I[Cold Storage @ $0.0046/GB-month];
10. H --> J[$11.28/month];
11. I --> K[$0.23/month];
12. J && K --> L[$11.51/month];
13. M[Network Cost] --> N1{Data Transfer};
14. N1 --> O[1 TB/day @ $0.09/GB];
15. O --> P[$93/month];
16. Q[Other Services] --> R1{Logging, Monitoring, Management Tools};
17. R1 --> S[$50/month + $75/month + $30/month];
18. S --> T[$155/month];
19. D && L && P && T --> U[$609.61/month];

