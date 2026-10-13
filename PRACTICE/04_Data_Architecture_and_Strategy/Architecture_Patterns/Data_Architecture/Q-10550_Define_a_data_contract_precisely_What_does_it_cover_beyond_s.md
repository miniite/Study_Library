---
type: "practice"
difficulty: "Expert"
---

# Define a data contract precisely. What does it cover beyond schema, how is it enforced in CI/CD, and what happens at runtime when a producer breaks it? Who is paged?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distinguishing Drift from Pipeline Breaks](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary TL;DR

In a data architecture, a comprehensive data contract covers more than just schema definitions; it includes constraints, policies, and business rules that ensure data quality, consistency, and integrity. Enforcing these contracts in CI/CD pipelines ensures that all changes meet the agreed-upon standards before they are deployed to production. At runtime, if a producer violates the contract, specific actions like logging, alerts, or even stopping the process can be triggered. The person responsible for monitoring the system is typically paged.

#### Triaging the Question

The question asks us to define a data contract precisely in the context of a data architecture (Lakes/Warehouses/Mesh). This involves understanding what elements are part of a complete data contract, how enforcement occurs during CI/CD processes, and the consequences when a producer breaches these contracts. The key aspects include schema definitions, business rules, data policies, and enforcement mechanisms.

#### First-Principles Solution

A data contract encompasses more than just the structure (schema) of the data. It includes:

Schema Definition: This outlines the columns, their types, and relationships.

#### Cons
traints: Ensuring that specific values or conditions are met. Examples include foreign key constraints, unique constraints, non-null requirements, etc.
Business Rules: These are logic-based rules governing how data should be manipulated and validated. For instance, ensuring that certain fields update based on others or that transactions maintain ACID properties.
Data Policies: Guidelines for how data should be handled in terms of access control, retention policies, and security.
Enforcement in CI/CD

Enforcing these contracts during the CI/CD pipeline ensures that any changes to the schema or logic are validated before deployment:

Schema Validation: Tools like Data ONTAP (Apache) or DBeaver can be used to validate new schemas against existing ones.
Unit Tests: Write unit tests in your CI/CD pipeline to check if the new code adheres to business rules and data policies.
Linters and Static Analysis: Use tools like ESLint for schema validation or SonarQube for code quality checks.
Code Reviews: Ensure that developers are reviewing each other's work to catch potential violations.
Runtime Enforcement and Consequences

At runtime, the system must be able to detect when a producer is violating the data contract:

Logging: Detailed logs should capture any deviations from expected behaviors.
Alerts: Implement alerting systems using tools like Prometheus or Grafana to notify relevant personnel.
Stopping Mechanisms: In severe cases, the system can halt further operations until the issue is resolved.
Who is Paged?

The person responsible for monitoring the data system should be paged when a contract violation occurs. This individual typically has roles such as a DevOps engineer, a Data Engineer, or a Site Reliability Engineer (SRE). They would receive alerts and need to investigate and resolve the issue promptly.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In conclusion, a comprehensive data contract is crucial for maintaining high standards of data quality. Enforcing this contract in CI/CD ensures that any changes are validated before deployment, while runtime enforcement mechanisms ensure immediate action when issues arise. The key roles involved include DevOps engineers, Data Engineers, and SREs.

#### Common Mistakes

Focusing only on schema definitions without including business rules and constraints.
Not implementing continuous validation in CI/CD pipelines.
Ignoring runtime enforcement and alerting mechanisms.
Triaging Follow-Ups
What tools would you recommend for enforcing data contracts?
How do you balance the complexity of enforcing complex business rules vs. maintaining developer productivity?
Can you walk me through a specific scenario where violating a contract caused a significant issue in your system?

**Comparison:**

```
Schema Definitions	Constraints	Business Rules	Data Policies
Name, type, fields	Foreign key	Field validation	Access control
```

**Code:**

```python
# Example CI/CD Pipeline Step for Schema Validation
import jsonschema
from flask import Flask, request

app = Flask(__name__)

@app.route('/validate-schema', methods=['POST'])
def validate_schema():
    schema_data = {
        "type": "object",
        "properties": {
            "field1": {"type": "string"},
            "field2": {"type": "integer"}
        },
        "required": ["field1", "field2"]
    }

    data = request.get_json()
    jsonschema.validate(data, schema_data)

    return "Schema validated successfully!", 200

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```text
Input:
{
  "name": "John Doe",
  "age": 30,
  "address": {
    "street": "123 Elm St",
    "city": "Springfield"
  }
}

Output:
{
  "message": "Schema validated successfully!"
}
```

**Data:**

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "User Profile Schema",
  "type": "object",
  "properties": {
    "name": {"type": "string"},
    "age": {"type": "integer"},
    "address": {"type": "object", "properties": {"street": {"type": "string"}, "city": {"type": "string"}}}
  },
  "required": ["name", "age"]
}
```

**Flow:**

1. Yes
2. No
3. CI/CD Pipeline
4. Validate Schema
5. Test and Linting
6. Deploy to Staging
7. Monitor in Prod
8. Violation?
9. Alert SRE
10. Production Deployment

