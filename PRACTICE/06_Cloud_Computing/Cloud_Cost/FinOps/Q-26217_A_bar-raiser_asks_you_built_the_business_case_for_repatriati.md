---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser asks: you built the business case for repatriating your steady-state workloads to bare metal and it penciled out, but two years later the cloud cut prices and released a cheaper instance generation. How did you account for cloud price deflation in the original decision, and would you reverse course?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Commitments and Spot Capacity](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Commitments_and_Spot_Capacity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In our original business case, we thoroughly analyzed cost drivers but underestimated the impact of cloud price fluctuations. While cloud providers often cut prices and introduce new generations, this should be a continuous part of our cost governance framework rather than a one-time consideration. We would need to reassess the total cost of ownership (TCO) now that prices have changed, and consider the long-term benefits of repatriating to bare metal versus staying in the cloud.

#### Triaging the Question

The core question revolves around how we account for unexpected changes in cloud pricing within our initial business case and whether those changes necessitate a reevaluation of our decision. The key points are:

Initial Cost Analysis: Our original cost governance model.
Price Deflation Impact: How recent price cuts by the cloud provider affect TCO.
Decision Reversal Consideration: Whether we should reverse course based on new information.

#### First-Principles Solution
Initial Cost Analysis

Baseline Calculation:

We estimated the total cost of running steady-state workloads in bare metal.
Factored in hardware costs, maintenance, and operational overhead.

Cloud Costs:

Assessed current cloud instance prices for comparison.
Estimated potential savings by moving to bare metal based on this comparison.

Cost Variability:

Acknowledged that cloud providers frequently adjust pricing and introduce new instance generations.
Accounted for these uncertainties in a best-case scenario but not as a primary driver.
Price Deflation Impact

New Instance Generation:

The cloud provider introduced a cheaper, more powerful generation of instances.
New prices reduced the cost of running workloads on cloud infrastructure significantly.

TCO Recalculation:

Re-evaluated TCO for both bare metal and cloud using updated pricing data.
Found that the newer cloud instance generation offers better value than our initial estimate.

Risk Assessment:

Evaluated long-term risks associated with repatriation, such as vendor lock-in and ease of scaling in the future.

#### Cons
idered potential future price changes; these could either mitigate or exacerbate cost savings.
Decision Reversal

Reassess TCO:

Recalculated TCO for both options: bare metal and cloud with new instance generation.
Found that staying in the cloud might now offer better cost efficiency over time, given recent price changes.

Future-Proofing:

Weighed future flexibility against initial hardware investments.
Discussed potential long-term savings from continued innovation in cloud technology vs. static bare metal costs.

Stakeholder Impact:

Communicated findings to stakeholders, highlighting both cost and operational benefits of different paths.
Recommended a hybrid approach: repatriate critical workloads where bare metal offers clear advantages but continue leveraging the cloud for flexibility and innovation.

#### Closing Summary

Given the recent price deflation in the cloud environment, it would be prudent to re-evaluate our initial decision. The updated TCO suggests that staying with the newer cloud instances could now offer better overall cost efficiency. However, we must also consider long-term strategic benefits of repatriation, such as hardware control and security.

#### Common Mistakes/Follow-Ups
Overlooking Future Price Fluctuations: Ignoring future price changes can lead to inaccurate TCO estimates.
Short-Term vs. Long-Term Decisions: Focusing solely on immediate savings without considering long-term strategic benefits.
Vendor Lock-In Risk: Assuming that cloud providers will always reduce prices, which might not be the case.

**Code:**

```python
# Example of a simple TCO calculation in Python
def calculate_tco(hardware_cost, maintenance_cost, cloud_instance_price, instance_hours):
    total_hardware_cost = hardware_cost + maintenance_cost
    cloud_total_cost = cloud_instance_price * instance_hours
    return (total_hardware_cost, cloud_total_cost)

# Initial estimates for bare metal and cloud
bare_metal_cost = calculate_tco(5000, 1200, None, 8760) # $5000 hardware + $1200 maintenance over a year

cloud_instance_price = 0.12  # Example price per hour for the original instance
new_cloud_instance_price = 0.08  # Updated price after deflation

cloud_cost_new_generation = calculate_tco(None, None, new_cloud_instance_price, 8760)
```

**Data:**

```json
{
    "bare_metal": {
        "hardware_cost": 5000,
        "maintenance_cost": 1200,
        "total_cost": null
    },
    "cloud_initial": {
        "instance_price": 0.12,
        "total_cost": 1051.2
    },
    "cloud_updated": {
        "instance_price": 0.08,
        "total_cost": 700.8
    }
}
```

