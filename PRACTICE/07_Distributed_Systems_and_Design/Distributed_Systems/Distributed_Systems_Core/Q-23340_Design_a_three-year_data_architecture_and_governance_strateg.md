---
type: "practice"
difficulty: "Expert"
---

# Design a three-year data architecture and governance strategy as the org grows from 5 to 50 services: ownership of shared entities, schema evolution, PII handling, a data contract mechanism between producers and consumers, and how you prevent the shared-database anti-pattern from creeping back in.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Federated vs Centralized Data Ownership](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Federated_vs_Centralized_Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In this scenario, we need to design a robust data architecture that supports growth from 5 to 50 services over three years. The key components include clear ownership of shared entities, schema evolution strategies, handling sensitive PII data, a contract-based system for producers and consumers, and preventive measures against the shared-database anti-pattern.

#### Triage
The core challenge here is ensuring data consistency, integrity, and security as the organization scales. We need to address three main areas: data ownership and governance, schema evolution, and handling sensitive data (PII). Additionally, we must implement a robust contract mechanism between producers and consumers to maintain consistent expectations about data structure and usage.

Step-by-Step Solution:

Ownership of Shared Entities: Define clear roles and responsibilities for each service. Use service-level agreements (SLAs) and an internal registry system to map shared entities and their owners. Each service should be responsible for its data but can request access from other services via an API.

Schema Evolution Strategy:

Version Control: Implement schema versioning using tools like Flyway or Liquibase. This ensures that when schemas change, all consuming services are aware of the changes and can handle them appropriately.
Automated Migrations: Use continuous integration (CI) pipelines to automate migrations so that new versions of a service do not break existing consumers.

Handling Sensitive PII Data:

Data Masking/Anonymization: Implement data masking techniques in the database and at the API layer to protect sensitive information.
Access Controls: Use role-based access control (RBAC) and least privilege principles to restrict access to PII.

Contract Mechanism Between Producers and Consumers:

API Design Guidelines: Establish a set of guidelines for APIs, including data types, validation rules, and response formats.
Documentation: Maintain comprehensive API documentation using tools like Swagger or Postman to ensure all services are aware of the contract.

Preventing Shared-Database Anti-Pattern:

Microservices Architecture: Ensure that each service has its own database, which is accessed via APIs. This prevents a single point of failure and simplifies maintenance.
Data Consistency Mechanisms: Implement distributed transaction protocols like Saga or compensation strategies to maintain data consistency across services.

#### Closing Summary
By implementing these strategies, we can ensure that our data architecture scales well while maintaining data integrity, security, and contract adherence. The key is to establish clear governance processes early on and to continuously refine them as the organization grows.

#### Common Mistakes/Follow-Ups

Shared Database Anti-Pattern: Many teams fall into this trap by starting with a shared database due to perceived simplicity but later realize the complexity and coupling it introduces.
Schema Evolution Management: Without proper tools and processes, schema changes can lead to data inconsistencies and service downtime.
Security Practices: Failing to implement robust security measures for sensitive data can result in compliance issues and data breaches.

**Comparison:**

```
Service	Owner Role	Shared Entities
User API	DataOps	User Profile
Order API	Finance	Payment Details
```

**Code:**

```python
# Example of a simple version control script using Flyway
def apply_schema_version(version):
    flyway = Flyway.configure().dataSource('<db_url>', '<username>', '<password>').load()
    flyway.migrate()
```

**Example input / output:**

| Input (Service Request) | Output (Response from Service) |
|-------------------------|--------------------------------|
| GET /users              | {"name": "John Doe", "email": "john@example.com"} |
| POST /orders            | {"id": 12345, "user_id": 67890} |

**Data:**

```yaml
api:
  - name: User API
    url: https://user-api.example.com/api/users
  - name: Order API
    url: https://order-api.example.com/api/orders
```

**Flow:**

1. Yes
2. No
3. User Service
4. Request API for Data
5. Is Data Shared?
6. Check Ownership and AccessControls
7. Fetch Data via API
8. Directly Fetch Data

