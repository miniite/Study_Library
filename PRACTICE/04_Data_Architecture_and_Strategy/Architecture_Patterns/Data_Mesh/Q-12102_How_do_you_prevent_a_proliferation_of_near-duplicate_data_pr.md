---
type: "practice"
difficulty: "Deep"
---

# How do you prevent a proliferation of near-duplicate data products across domains solving the same problem slightly differently?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Ownership vs Cross-cutting Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Domain_Ownership_vs_Cross-cutting_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Data Mesh architecture, ensuring that there is no proliferation of near-duplicate data products across domains is crucial to maintaining consistency and efficiency in data usage. The key is to leverage a well-defined set of principles and practices while allowing autonomy among domain teams. Let's break down the solution from first principles.

Firstly, the question is essentially probing how we can maintain data consistency and avoid redundancy when different domains are responsible for creating similar data products. We need to ensure that there is a clear understanding of what constitutes "near-duplicate" data products, establish governance mechanisms, and implement robust standards and practices.

#### Assumptions

Each domain has its own set of responsibilities but needs to collaborate effectively.
The goal is to avoid unnecessary duplication while still allowing for flexibility in how data is processed and used within each domain.
We need a mechanism to define data ownership and stewardship roles clearly.
There should be a standardized process for validating and merging near-duplicate products.
Step-by-Step Solution

Data Ownership and Stewardship:

Each domain must have clear ownership of the data they produce, ensuring that no single entity is responsible for all domains’ data. This promotes domain-specific expertise.

Domain-Specific Data Products:

Define what constitutes a near-duplicate product within each domain. For example, if multiple domains are handling customer data but in slightly different ways, ensure they share common metadata and validation checks.

Standardization of Data Definitions:

Develop standardized definitions for critical fields (e.g., customer ID, transaction type) across all domains to reduce confusion and ensure consistency.

Governance Frameworks:

Implement a governance framework that includes regular reviews of data products by cross-domain committees or data stewards. This helps in identifying and resolving near-duplicates early.

Data Validation and Merging Processes:

Establish a process for validating new data products against existing ones. Tools like ETL validation, schema comparison, and automated testing can be used to ensure that the new product is not just similar but identical where necessary.

Cross-Domain Collaboration:

Foster an environment where domain teams can collaborate on shared goals without losing their autonomy. This could involve regular cross-domain meetings or a central data portal for sharing insights.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, preventing near-duplicate data products in a Data Mesh architecture requires a balance between domain independence and enterprise-wide consistency. Key steps include clear ownership, standardization of definitions, robust governance frameworks, and processes for validating and merging new products. Common pitfalls include overly centralized control, lack of collaboration, and insufficient validation mechanisms.

A typical follow-up might be asking about specific tools or technologies that can support these practices, such as data cataloging platforms or automated ETL validation systems.

**Comparison:**

```
Input Schema 1	Input Schema 2	Differences Detected
customer_id
name
email
phone	-	Field 'phone' is missing in schema 2.
```

**Code:**

```python
# Example code for basic schema comparison
def compare_schemas(schema1, schema2):
    differences = []
    for field in schema1:
        if field not in schema2:
            differences.append(f"Field {field} is missing in schema 2.")
    for field in schema2:
        if field not in schema1:
            differences.append(f"Field {field} is missing in schema 1.")
    return differences
```

**Example input / output:**

| Input Schema 1 | Input Schema 2 | Differences Detected |
|----------------|----------------|----------------------|
| customer_id    |                |                      |
| name           |                |                      |
| email          |                |                      |
| phone          | -              | Field 'phone' is missing in schema 2. |

**Data:**

```json
{
  "schema1": ["customer_id", "name", "email", "phone"],
  "schema2": ["customer_id", "name", "email"]
}
```

**Flow:**

1. Define Data Ownership
2. Implement StandardDefinitions
3. Establish GovernanceFrameworks
4. Set Up Validation Processes
5. Foster Cross-DomainCollaboration
6. Monitor and Improve

