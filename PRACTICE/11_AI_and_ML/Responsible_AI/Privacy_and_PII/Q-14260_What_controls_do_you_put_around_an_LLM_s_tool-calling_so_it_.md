---
type: "practice"
difficulty: "Deep"
---

# What controls do you put around an LLM's tool-calling so it can query a customer DB for a legitimate task but can't be coaxed into dumping other customers' PII?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ PII Filters for LLMs](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/PII_Filters_for_LLMs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a Large Language Model (LLM) can perform necessary queries on a customer database while preventing it from exposing Personal Identifiable Information (PII), we need to implement several robust controls and design principles. The core idea is to limit the LLM's access, monitor its interactions, and enforce strict data handling policies.

#### Triage
The question is probing into how to balance functionality and security in an AI system that interacts with sensitive customer databases. We must ensure that the LLM can only query specific, authorized information without exposing other customers' data. Key assumptions include:

The database contains PII (e.g., names, addresses, credit card numbers).
The task involves legitimate queries for a specific user.
There are mechanisms to authenticate and authorize users.

#### First-Principles Solution

Data Segmentation: Divide the database into segments based on customer ID or other unique identifiers to limit access.
Role-Based Access Control (RBAC): Implement RBAC to restrict which queries can be executed by the LLM.
Query Validation: Ensure all incoming requests are validated against a predefined set of allowed queries.
Logging and Monitoring: Maintain detailed logs to detect any unauthorized or suspicious activity.

Let's break down each control in detail:

Data Segmentation:

Segment the database such that only the relevant customer data is accessible based on their unique identifier.

#### Example
schema:
SQL
CREATE TABLE CustomerData (
  customerId INT PRIMARY KEY,
  name VARCHAR(255),
  address VARCHAR(255),
  creditCardNumber VARCHAR(16)
);

Role-Based Access Control (RBAC):

Define roles that the LLM can assume based on its task.
For example, a role might be "queryCustomer" which allows access to specific fields for a particular customer.

#### Example
policy:
JSON
{
  "roles": [
    {
      "name": "queryCustomer",
      "permissions": ["select name, address from CustomerData where customerId = '123456'"
    }
  ]
}

Query Validation:

Ensure that all queries sent to the database are validated against a set of predefined templates.

#### Example
validation function in Python:
PYTHON
def validate_query(query):
    allowed_fields = ['name', 'address']
    if not all(field in allowed_fields for field in query.split()):
        raise ValueError("Invalid fields in query")
    return True

Logging and Monitoring:

Maintain detailed logs of every interaction between the LLM and the database.
Use monitoring tools to detect any unauthorized or suspicious activity, such as excessive queries or attempts to access forbidden data.

#### Example
logging setup:
PYTHON
import logging

def log_query(query, response):
    logger = logging.getLogger(__name__)
    logger.info(f"Query: {query}, Response: {response}")

#### Closing Summary
and Common Mistakes/Follow-Ups: By implementing these controls, we can ensure that the LLM is restricted to performing legitimate tasks without exposing sensitive PII. The key challenges are ensuring data segmentation is granular enough, validating queries accurately, and monitoring for any potential breaches.

#### Common mistakes
include:

Failing to segment data properly, leading to accidental exposure of PII.
Overly permissive query validation rules, allowing unauthorized access.
Insufficient logging or lack of real-time monitoring mechanisms.

For follow-ups, we might discuss how to handle edge cases like multi-step queries or the integration with other security tools.

**Code:**

```python
def validate_query(query):
    allowed_fields = ['name', 'address']
    if not all(field in allowed_fields for field in query.split()):
        raise ValueError("Invalid fields in query")
    return True

def log_query(query, response):
    logger = logging.getLogger(__name__)
    logger.info(f"Query: {query}, Response: {response}")
```

