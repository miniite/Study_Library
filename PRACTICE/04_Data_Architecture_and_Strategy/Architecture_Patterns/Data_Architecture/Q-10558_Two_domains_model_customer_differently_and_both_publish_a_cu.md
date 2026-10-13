---
type: "practice"
difficulty: "Expert"
---

# Two domains model 'customer' differently and both publish a 'customer' data product. As the principal, how do you resolve the semantic conflict without imposing a central data model that nobody uses?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-domain Joins in a Mesh](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Cross-domain_Joins_in_a_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the semantic conflict between two domains publishing a 'customer' data product, we need to design a robust data architecture that respects each domain's unique requirements while ensuring seamless integration and minimal centralization. The key idea is to leverage a decoupled approach with separate data warehouses, combined with a service mesh or API gateway for mediation.

Firstly, let’s break down the problem: Domain A models customers in one way, and Domain B models them differently. Both domains need to publish customer data products independently but must still maintain consistency when integrated across the organization. The challenge is to resolve this without enforcing a single, monolithic model that might not align with either domain's needs.

确认答案涵盖了所有关键点，且内容完整准确。

The first step involves understanding each domain’s specific customer data requirements and how they differ. For example:

Domain A may focus on detailed transactional records.
Domain B might prioritize demographic information for marketing purposes.

These differences must be respected in their respective data warehouses to maintain fidelity with the original models.

Next, we introduce a service mesh or API gateway that acts as an intermediary layer. This architecture allows each domain to maintain its own data model while ensuring that customer data is consistently accessible across different use cases. The service mesh can:

Data Transformation: It can transform data from one schema to another when necessary, ensuring that customers in both domains are represented uniformly where required.
API Layering: By exposing APIs, the service mesh enables seamless integration of data products without forcing a single model.
Governance and Security: The mesh provides a centralized point for managing security policies, access controls, and compliance requirements.

Here’s an example using a service mesh to illustrate this concept:

Publish
Publish
Transform/Access Control
Transform/Access Control
Domain A
Service Mesh 1
Domain B
Service Mesh 2
Customer API Gateway

In this diagram, AMesh1 and BMesh2 handle the internal logic of each domain. The CustomerAPI serves as a unified interface for accessing customer data, transforming it if necessary to meet organizational standards.

**Comparison:**

| Domain A Schema | Domain B Schema       | Unified Schema |
|-----------------|----------------------|----------------|
| customer_id     | id                   | id             |
| name            | full_name            | name           |
| email           | n/a                  | email          |
| transactions    | transaction_history  | transaction_history |

**Code:**

```python
# Example Python code snippet for service mesh API transformation

def transform_customer_data(customer):
    # Example transformation based on target schema
    transformed = {
        "id": customer["customer_id"],
        "name": customer.get("name"),
        "email": customer.get("email"),
        "transaction_history": customer["transactions"]
    }
    return transformed
```

**Example input / output:**

```text
Input:
Domain A customer data product (in their own schema)
{
  "customer_id": 12345,
  "name": "John Doe",
  "transactions": [
    {"date": "2023-10-01", "amount": 150.00},
    {"date": "2023-10-05", "amount": 200.00}
  ]
}

Output:
Unified customer data product (transformed and consistent)
{
  "id": 12345,
  "name": "John Doe",
  "email": null,  # Demographic not available
  "transaction_history": [
    {"date": "2023-10-01", "amount": 150.00},
    {"date": "2023-10-05", "amount": 200.00}
  ]
}
```

**Data:**

```json
{
  "customer": {
    "customer_id": 12345,
    "name": "John Doe",
    "email": "johndoe@example.com",
    "transactions": [
      {"date": "2023-10-01", "amount": 150.00},
      {"date": "2023-10-05", "amount": 200.00}
    ]
  }
}
```

**Flow:**

1. Publish
2. Publish
3. Transform/Access Control
4. Transform/Access Control
5. Domain A
6. Service Mesh 1
7. Domain B
8. Service Mesh 2
9. Customer API Gateway

