---
type: "practice"
difficulty: "Expert"
---

# How would you build a data-quality strategy that's enforced at the platform level (not per-team) without every team hating the gates — where do you put contracts vs tests vs monitoring?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Platform-level Data Quality](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Platform-level_Data_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build an effective data-quality strategy enforced at the platform level, we need to balance stringent governance with flexibility and collaboration across teams. This involves designing a platform that allows teams to work within agreed-upon quality standards without feeling overly constrained. The core idea is to define contracts, tests, and monitoring mechanisms in a way that minimizes friction while ensuring data integrity.

The key components of the strategy include:

Defining Contracts: These are declarative statements about expected data qualities such as schema constraints, format validation rules, and business rule enforcement.
Implementing Tests: Automated checks to verify that data conforms to the contracts during ingestion and processing.
Monitoring Mechanisms: Continuous monitoring tools to alert stakeholders of potential issues before they impact downstream processes.

Let's break this down step-by-step from first principles:

#### Triaging the Question

The interviewer is looking for a comprehensive strategy that ensures data quality at the platform level without alienating individual teams. This requires understanding the trade-offs between centralization and decentralization, as well as considering how to balance governance with operational flexibility.

Defining Contracts

Contracts are essential because they provide clear expectations about what constitutes good-quality data. They should be:

Declarative: Specify what is allowed or disallowed without prescribing a specific implementation.
Inter-team Agnostic: Clear enough for any team to understand and adhere to, regardless of the specific tools or processes used.

#### Example
s of contracts might include:

Schema Validation: Ensuring that columns have the correct types (e.g., all dates are in ISO 8601 format).
Range Checks: Validating that values fall within expected ranges (e.g., age > 0).
Implementing Tests

Tests are used to enforce these contracts during data ingestion and processing. They should:

Be Automated: Run automatically as part of the ETL pipeline.
Be Efficient: Minimize overhead but still catch critical issues.

#### Example
s of tests might include:

Schema Validations: Using libraries like Apache Avro or JSON Schema validation tools.
Data Type Checks: Ensuring consistent data types across columns (e.g., using SQL constraints).
Monitoring Mechanisms

Monitoring is crucial for catching and addressing quality issues before they escalate. This should involve:

Real-time Alerts: Notifying teams of potential issues immediately.
Historical Reporting: Providing historical views to understand trends over time.

#### Example
s of monitoring tools might include:

Data Quality Dashboards: Using tools like Looker or Tableau to visualize data quality metrics.
Logging and Auditing: Keeping detailed logs for future reference.
Placing Contracts vs Tests vs Monitoring
Contracts should be defined centrally but with clear guidelines for teams to follow. These can be embedded in documentation, code repositories, or metadata stores.
Tests are best placed within the data ingestion pipelines where they can enforce contracts as soon as data enters the system.
Monitoring mechanisms should span both platform and team levels. Platform-level monitoring ensures overall health, while team-specific monitoring helps individual teams stay aware of their contributions.

#### Closing Summary

By centralizing governance through clear contracts, implementing robust tests at key points in the pipeline, and providing comprehensive monitoring, we can build a data-quality strategy that respects individual team autonomy while ensuring platform-wide integrity. This approach minimizes friction by aligning with business needs and leveraging existing tools and practices.

#### Common Mistakes

Overly restrictive contracts that stifle creativity.
Poor test coverage leading to late-stage data issues.
Inadequate monitoring causing delayed awareness of problems.

**Comparison:**

```
Contract Type	Description
Schema	Defines valid column types and ranges
Data Integrity	Ensures consistent data values
Business Rules	Enforces domain-specific constraints
```

**Code:**

```python
# Example Python code for schema validation using jsonschema library
import jsonschema

def validate_schema(data, schema):
    try:
        jsonschema.validate(instance=data, schema=schema)
        return True
    except jsonschema.exceptions.ValidationError as err:
        print(f"Validation error: {err}")
        return False
```

**Example input / output:**

```text
Input Data:
{
  "name": "John Doe",
  "age": -30
}

Output (Validation Error):
Validation error: -30 is less than the minimum of 18 for age.
```

**Data:**

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "name": {"type": "string"},
    "age": {"type": "integer", "minimum": 18}
  },
  "required": ["name", "age"]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Ingestion
6. Is the data schema valid?
7. Execute Data Transformation
8. Log Error and Reject Data
9. Maintain Historical Reports
10. Notify Stakeholders
11. Is there a business ruleviolation?
12. Alert Team Lead
13. Monitor for Future Issues

