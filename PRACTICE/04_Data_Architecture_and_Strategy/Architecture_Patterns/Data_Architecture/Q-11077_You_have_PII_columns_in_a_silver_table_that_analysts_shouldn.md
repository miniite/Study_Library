---
type: "practice"
difficulty: "Deep"
---

# You have PII columns in a silver table that analysts shouldn't see unmasked. How would you architect masking so it works whether they query via Spark or the warehouse, without maintaining two copies?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Proving PII Residency](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Proving_PII_Residency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of ensuring PII (Personally Identifiable Information) columns are masked when queried by analysts, we need to design a robust data architecture that works seamlessly across different query methods—Spark and the data warehouse. The key challenge is to avoid maintaining two copies of the data while ensuring compliance with data masking policies. This can be achieved using a combination of database-level encryption, transparent data encryption (TDE), row-level security (RLS), and virtualization techniques.

#### Triage
What We're Probing

The core question here involves understanding how to implement effective data masking that is both robust and flexible. The interviewer will likely probe on the following points:

Masking Techniques: How do we mask PII without exposing sensitive information?
Cross-Platform Support: Ensuring the solution works seamlessly with Spark and the warehouse.

#### Performance Considerations
Balancing performance with security constraints.
Compliance and Governance: Ensuring compliance with data protection regulations.

#### Assumptions
We are using a modern data architecture that includes both on-premises and cloud-based solutions.
The data is stored in an analytics warehouse (e.g., Redshift, BigQuery).
The analysts use Spark for querying the data via an interactive or batch processing framework.

#### First-Principles Solution
Transparent Data Encryption (TDE): Encrypt PII columns at rest and in transit to ensure data is protected even if it leaves the database.
Row-Level Security (RLS): Implement RLS on the warehouse side to restrict access based on user roles or permissions, ensuring analysts can only see masked data.
Masking Layer: Use a masking layer that applies transformations before querying, ensuring PII is not visible in its original form.
Virtual Tables and Views: Create virtual tables/views that are transparently masked via the masking layer.
Step-by-Step Implementation

Database Encryption:

Enable TDE on your data warehouse to encrypt sensitive columns at rest and in transit. This ensures that even if the data is compromised, it remains unreadable.
Use database-level encryption for PII fields to prevent unauthorized access or exposure.

Row-Level Security (RLS):

Implement RLS policies that restrict read access based on user roles. For example, create a policy where only certain roles can see unmasked data.
Define these policies in the warehouse metadata to ensure they are applied consistently across different queries.

Masking Layer:

Develop a middleware layer that intercepts queries and applies transformations before passing them to the warehouse.
Use a masking strategy such as partial masking (e.g., truncating or replacing parts of the PII), scrambling, or tokenization.
The masking layer can be implemented using a combination of stored procedures, user-defined functions, or a dedicated service that handles data transformation.

Virtual Tables and Views:

Create virtual tables/views in the warehouse that are transparently masked by the middleware layer.
Ensure these views return only the transformed (masked) data, which is safe for analysts to query without exposing PII.

Query Proxying:

Use a query proxy or middleware service to intercept Spark queries before they reach the warehouse.
The proxy can dynamically apply masking rules based on the user and query context.

#### Closing Summary

By combining TDE, RLS, a dedicated masking layer, and virtual tables/views, we can ensure that PII is masked appropriately when queried by analysts via both Spark and the warehouse. This approach maintains data security while enabling flexible access through different querying mechanisms.

#### Common Mistakes/Follow-Ups
Maintaining Two Copies: Avoiding maintaining two copies of the data is crucial to prevent inconsistencies.
Performance Impact: Be aware that additional layers can impact performance, so monitoring and optimizing the masking layer is essential.
User Experience: Ensure that analysts do not experience significant delays or complications due to the added complexity of masked queries.

#### Example
Code Snippet
PYTHON
# Example Python code for a simple masking function
def mask_pii(data):
    if 'PII_COLUMN' in data:
        # Apply partial masking (e.g., truncating last 4 characters)
        masked_data = data[:len(data) - 4] + "****"
        return masked_data
    else:
        return data

# Example usage
original_data = {'name': 'John Doe', 'email': 'johndoe@example.com'}
masked_data = mask_pii(original_data)
print(masked_data)

**Comparison:**

```
Column	Masking Strategy
name	partial_mask
email	tokenize
```

**Code:**

```python
def mask_pii(data):
    if 'PII_COLUMN' in data:
        masked_data = data[:len(data) - 4] + "****"
        return masked_data
    else:
        return data

original_data = {'name': 'John Doe', 'email': 'johndoe@example.com'}
masked_data = mask_pii(original_data)
print(masked_data)
```

**Example input / output:**

```text
Input:
original_data = {'name': 'John Doe', 'email': 'johndoe@example.com'}

Output:
{'name': 'Jo****Doe', 'email': '[TOKENIZED_EMAIL]'}
```

**Data:**

```json
{
  "PII_COLUMNS": ["name", "email"],
  "MASKING_LAYER_RULES": {
    "name": "partial_mask",
    "email": "tokenize"
  }
}
```

**Flow:**

1. Yes
2. No
3. Query Proxy/Spark
4. Is Analyst?
5. Masking Layer
6. Warehouse
7. Warehouse Direct Query

