---
type: "practice"
difficulty: "Deep"
---

# How do you decide whether data mesh, data fabric, or a plain modern warehouse is the right answer for a mid-size company; what actually differentiates them in practice?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Mesh Principles](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Mesh_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between data mesh, data fabric, and a traditional modern data warehousing solution for a mid-sized company, it's crucial to understand the fundamental principles of each approach. Data mesh emphasizes decentralization, self-service, and domain-driven design, whereas data fabric offers a more integrated and flexible architecture. A plain modern warehouse is a simpler, centralized model that focuses on consolidating all data into one location.

The key differentiators lie in the business requirements, technical capabilities, and organizational structure of the mid-sized company. We need to carefully evaluate these factors to determine which approach aligns best with the company's goals.

**Comparison:**

| Approach           | Flexibility   | Integration Complexity | Scalability       |
|--------------------|---------------|------------------------|-------------------|
| Data Mesh          | High          | Medium-High             | Good              |
| Data Fabric        | Medium        | Low                    | Excellent         |
| Modern Warehouse   | Low           | High                   | Very Good         |

**Code:**

```python
# Example code snippet for data processing in a modern warehouse
def process_data(data):
    transformed_data = transform(data)
    return store(transformed_data)

# Placeholder functions
def transform(raw_data):
    # Transformation logic
    pass

def store(processed_data):
    # Storage logic
    pass
```

**Example input / output:**

```text
Given the company profile and data sources, here’s a hypothetical input-output example for evaluating different approaches:

- **Input**: Requirements to integrate customer feedback with sales data.
- **Output**:
    - Data Mesh: Separate domains handle their own data, ensuring flexibility but increasing complexity in integration.
    - Data Fabric: Provides a unified view while enabling domain-specific processing, balancing flexibility and integration.
    - Modern Warehouse: Centralizes all data, simplifying the analysis process but requiring robust ETL processes.

| Approach           | Flexibility   | Integration Complexity | Scalability       |
|--------------------|---------------|------------------------|-------------------|
| Data Mesh          | High          | Medium-High             | Good              |
| Data Fabric        | Medium        | Low                    | Excellent         |
| Modern Warehouse   | Low           | High                   | Very Good         |
```

**Data:**

```json
{
  "company_profile": {
    "industry": "Retail",
    "size": "Mid-size (50-250 employees)",
    "goals": ["Improve customer insights", "Enhance operational efficiency"],
    "technical_stack": ["SQL databases", "NoSQL databases"]
  },
  "data_sources": [
    {"type": "Customer feedback", "source": "CRM system"},
    {"type": "Sales data", "source": "ERP system"}
  ]
}
```

**Flow:**

1. Data Source
2. ETL Process
3. Modern Warehouse
4. Analysis
5. Domain-Specific Processing
6. Data Fabric
7. Unified View
8. Self-Service Domains
9. Data Mesh
10. Integrated Access

