---
type: "practice"
difficulty: "Deep"
---

# What is your reference architecture for a bronze-silver-gold medallion layout of an observability lake and how do you decide what earns promotion to gold?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Curated Observability Datasets](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Curated_Observability_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an observability data lake with a bronze-silver-gold (BSG) medallion layout, we aim to create a robust architecture that ensures data is stored in different levels of quality and accessibility, from raw, unprocessed "bronze" data all the way up to refined "gold" insights. The key principle here is to ensure that each level serves its purpose while enabling seamless data flow for analysis at higher levels.

The BSG layout essentially involves three distinct layers:

Bronze Layer: Raw event data collected from various sources, often unstructured and not yet cleaned or transformed.
Silver Layer: Processed and validated data, ready to be consumed by downstream systems like dashboards and alerting mechanisms.
Gold Layer: Highly refined, aggregated, and enriched data suitable for complex analytics, machine learning models, and long-term storage.

In this architecture, the decision on what data gets promoted from one layer to another is driven by factors such as data quality, timeliness, and business need. Let's break down each step in detail.

**Comparison:**

| Level  | Description                             |
|--------|-----------------------------------------|
| Bronze | Raw event data                          |
| Silver | Processed, validated                    |
| Gold   | Highly refined, aggregated insights     |

**Code:**

```python
# Example Python pseudocode for BSG promotion logic
def promote_to_silver(bronze_data):
    # Data validation, transformation, and enrichment
    silver_data = transform_and_enrich(bronze_data)
    return silver_data

def enrich_silver_data(silver_data):
    # Further transformations, aggregations, and enrichment
    gold_data = perform_aggregations_and_enrichments(silver_data)
    return gold_data

def promote_to_gold(gold_data):
    # Final refinement and storage in a highly available manner
    store_in_gold_storage(gold_data)
```

**Example input / output:**

```text
# Example input/output flow
Input:
- Raw event logs (bronze layer)

Output:
1. Processed and validated data -> Silver Layer
2. Aggregated, enriched insights -> Gold Layer

Example dry-run:
Raw Logs -> Bronze Layer
  |-> Data Validation & Enrichment -> Silver Layer
  |   |-> Aggregation & Enrichment -> Gold Layer
```

**Data:**

```yaml
# Example data schema for BSG layers
observability_data:
  - type: bronze
    description: Raw event logs from various sources.
  - type: silver
    description: Processed, validated, and enriched for immediate use.
  - type: gold
    description: Highly refined insights used for complex analysis and long-term storage.
```

**Flow:**

1. Raw Logs
2. Bronze Layer
3. Silver Layer
4. Gold Layer

