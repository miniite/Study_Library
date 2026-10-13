---
type: "practice"
difficulty: "Deep"
---

# How do you architect a 'data product SLO' framework distinct from data quality: what availability, freshness, and correctness SLIs do you standardize across a mesh, how do you set error budgets on data products, and what does an SLO breach trigger operationally?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product SLAs](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Data_Product_SLAs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a 'data product Service Level Objective (SLO) framework' distinct from data quality in a distributed mesh architecture, we need to focus on availability, freshness, and correctness SLIs. These SLOs ensure that our data products meet specific reliability and timeliness targets. I'll outline the key steps and considerations for setting up these SLIs, defining error budgets, and operational responses.

#### Triage
The question is asking how to define and implement Service Level Objectives (SLOs) specifically tailored to data products in a mesh architecture, distinguishing them from traditional data quality metrics. We need to understand the service level indicators (SLIs), set error budgets, and plan operational actions for breaches.

Key Assumptions:

The mesh architecture consists of multiple interconnected data sources and processing pipelines.
Data products are defined as specific use cases or queries leveraging this mesh.
SLIs will be standardized across these products to ensure consistency in measurement.
Error budgets will be set based on the impact and criticality of each product.

Step-by-Step Solution:

Define SLIs for Availability, Freshness, and Correctness:

Availability (A): The percentage of time a data product is available to users. [ A = 1 - \frac{\text{Number of Outages}}{\text{Total Time}}

#### Example
For 99.9% availability, the system should be unavailable for no more than 43 minutes in a year.

Freshness (F): The freshness SLI ensures data is up-to-date. [ F = \frac{\text{Number of Recent Data Updates}}{\text{Total Number of Data Entries}}

#### Example
For real-time stock data, this might be 100% within a few seconds.

Correctness (C): The correctness SLI ensures the accuracy and reliability of the data. [ C = 1 - \frac{\text{Number of Inconsistent Data Entries}}{\text{Total Number of Data Entries}}

#### Example
For financial records, this might be 99.5% to account for occasional discrepancies.

Set Error Budgets:

Define the allowable error rate for each SLI based on business and operational requirements.

#### Example
Availability: 0.1% (1 hour of downtime per year)
Freshness: 0.5% (5 minutes out-of-sync with source)
Correctness: 0.25%

Operational Responses for SLO Breaches:

Availability: Implement retries, fallback mechanisms, and failover strategies to maintain service.
Freshness: Develop real-time monitoring and alerting systems to detect delays in data processing.
Correctness: Set up validation processes using checksums, hashes, or consistency checks.

Standardization Across Mesh:

Create a common metrics dashboard for all data products to monitor SLOs consistently.
Use standardized alerts and notifications to trigger actions automatically when thresholds are breached.

#### Example
Slack notifications, PagerDuty incidents, or cloud-based monitoring services like Datadog.

#### Closing Summary
By defining clear SLIs for availability, freshness, and correctness, setting robust error budgets, and establishing operational responses, we can ensure that our data products meet specific reliability and timeliness targets. This approach provides a structured way to manage the quality of data products in a complex mesh architecture while distinguishing it from broader data quality metrics.

#### Common Mistakes/Follow-Ups

Ignoring the importance of real-time monitoring for freshness.
Overlooking the need for validation mechanisms to ensure correctness.
Failing to implement automated responses that can mitigate breaches proactively.
Not standardizing metrics and alerts across all products, leading to inconsistent practices.

**Comparison:**

```
SLI	Definition	Example Value
A	The percentage of time a data product is available to users	99.9% (43 mins)
F	The freshness of the data, ensuring it is up-to-date	100% (5 mins)
C	The correctness of the data, ensuring accuracy and reliability	99.75%
```

**Code:**

```python
# Example Python code for calculating SLIs
def calculate_availability(outages):
    total_time = 365 * 24 * 60 * 60  # Total time in a year (in minutes)
    availability = 1 - (outages / total_time)
    return availability

def calculate_freshness(recent_updates, total_entries):
    freshness = recent_updates / total_entries
    return freshness

def calculate_correctness(inconsistent_entries, total_entries):
    correctness = 1 - (inconsistent_entries / total_entries)
    return correctness
```

**Example input / output:**

```text
Input:
- Outages: 43 minutes in a year (for 99.9% availability)
- Recent Updates: 99,850 entries out of 100,000
- Inconsistent Entries: 250

Output:
- Availability: 99.9%
- Freshness: 99.85%
- Correctness: 99.75%
```

**Data:**

```json
{
  "outages": 43,
  "recent_updates": 99850,
  "total_entries": 100000,
  "inconsistent_entries": 250
}
```

**Flow:**

1. Availability SLI Breach
2. Retries/Failover
3. Freshness SLI Breach
4. Real-time Monitoring
5. Notify DevOps
6. Correctness SLI Breach
7. Validation Check

