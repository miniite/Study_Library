---
type: "practice"
difficulty: "Deep"
---

# What is your framework for deciding which telemetry gets a strict typed schema versus schema-on-read flexibility given hundreds of heterogeneous vendor log sources?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Exactly Once into Observability Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Exactly_Once_into_Observability_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between strict typed schemas and schema-on-read flexibility for telemetry data from hundreds of heterogeneous vendors, the goal is to balance consistency, maintainability, and the ability to handle unexpected or evolving data formats. Our framework considers several key factors, including data stability, vendor support, operational complexity, and long-term maintenance.

First, we need to understand that strict typed schemas offer strong guarantees about data structure and are suitable for well-defined, stable telemetry sources from reliable vendors. Conversely, schema-on-read flexibility allows us to handle a wide variety of data formats dynamically but can introduce complexities in processing and analysis. Let’s break down the decision-making process step by step.

#### Triaging the Question

The question is probing our understanding of how to manage diverse data sources effectively while ensuring data integrity and usability. We need to consider assumptions such as the data's predictability, the frequency of changes, and the tools we will use for processing and analyzing this data. We also assume that the goal is to maintain a robust observability platform that can adapt to both known and unknown data formats.

#### First-Principles Solution
Data Stability: Evaluate the stability of each vendor's telemetry source. Sources with highly stable data patterns, like logs from well-maintained infrastructure components (e.g., network devices), are better suited for strict typed schemas.
Vendor Support and Reliability: Consider the reliability of vendors providing the telemetry. Established vendors with consistent practices may be easier to handle using strict schemas due to their predictability.
Processing Complexity: Assess the complexity introduced by each data source in terms of parsing, validation, and integration into our observability platform. Sources that introduce minimal processing overhead are more attractive for typed schemas.
Operational Impact: Consider how changes in telemetry format will impact operations. Frequent or unpredictable schema changes should push us towards schema-on-read flexibility to minimize downtime and rework.
Decision Framework

Highly Stable Telemetry:

Use strict typed schemas if the data is known, stable, and from reliable vendors.

#### Example
Network device logs with consistent fields over time.

Unstable or Evolving Data:

Favor schema-on-read flexibility for sources that frequently change their format or are less predictable (e.g., application logs).

#### Example
Application log output which can vary based on deployment environment or runtime conditions.

New Vendor Integration:

Initially use schema-on-read to quickly integrate and validate data.
Transition to typed schemas once patterns emerge and stabilize over time.

#### Example
Integrating a new SaaS application whose log format may evolve initially but will likely settle down eventually.

Complex Data Processing:

Schema-on-read is beneficial for reducing initial setup costs and handling complex, irregular data structures.

#### Example
User activity logs where the fields might vary widely across different user sessions or devices.

#### Closing Summary

In summary, our framework prioritizes stability, vendor reliability, processing complexity, and operational impact. We use strict typed schemas for highly stable, predictable sources and schema-on-read flexibility for unstable, evolving data or new integrations. By carefully considering these factors, we can create a robust observability platform that efficiently manages diverse telemetry data while minimizing operational overhead.

#### Common Mistakes/Follow-Ups
Overlooking Vendor Support: Failing to assess the reliability of vendors may lead to unexpected changes in data format.
Ignoring Operational Impact: Not accounting for how schema changes will impact operations can result in frequent downtime and rework.
Inconsistent Decision-Making: Applying a blanket approach without considering individual source characteristics can lead to inefficiencies.

**Comparison:**

```
Data Source	Stability (%)	Vendor Reliability (0-10)	Processing Complexity (1-10)	Schema Type
NetworkDevice	95	8.2	3	typed
ApplicationLog	40	6.5	7	on-read
```

**Code:**

```python
# Example of a Python function to decide on schema type based on telemetry data stability
def decide_schema_type(stability, vendor_reliability, processing_complexity):
    if stability > 80 and vendor_reliability >= 7:
        return "typed"
    elif processing_complexity < 5 or vendor_reliability < 3:
        return "on-read"
    else:
        return "review"  # Further analysis needed
```

**Example input / output:**

```text
Input:
{
  "data_sources": [
    {"name": "NetworkDevice", "stability": 95, "vendor_reliability": 8.2, "processing_complexity": 3},
    {"name": "ApplicationLog", "stability": 40, "vendor_reliability": 6.5, "processing_complexity": 7}
  ]
}

Output:
{
  "NetworkDevice": "typed",
  "ApplicationLog": "on-read"
}
```

**Data:**

```json
{
  "data_sources": [
    {
      "name": "NetworkDevice",
      "stability": 95,
      "vendor_reliability": 8.2,
      "processing_complexity": 3
    },
    {
      "name": "ApplicationLog",
      "stability": 40,
      "vendor_reliability": 6.5,
      "processing_complexity": 7
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Telemetry Source
6. Stable?
7. Strict Typed Schema
8. Frequent Changes?
9. Schema-on-Read Flexibility
10. Review and Decide

