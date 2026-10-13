---
type: "practice"
difficulty: "Deep"
---

# How would you evaluate whether to give the AI direct DB/read access via tools versus keeping it text-only, weighing capability against blast radius?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing AI Code for Performance Risks](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Performance_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether to grant direct database read access to an AI like Copilot/Cursor, we need to balance the potential benefits of enhanced coding capabilities with the risk of accidental data exposure or misuse. This decision involves considering factors such as data sensitivity, project complexity, and the maturity of the AI tool.

The core question here is: How do you weigh the advantages of direct database access against the potential risks? Let's break this down step by step from first principles.

#### Triage
Understanding the Question: The interviewer is probing your ability to assess risk versus utility, and your understanding of data management practices in a collaborative coding environment.

#### Assumptions
We are working on a project where AI-assisted coding tools like Copilot/Cursor might be used.
The database contains sensitive or critical information that could potentially be misused if accessed improperly.

#### First-Principles Solution
1. Assess Data Sensitivity and Use Cases

First, classify the data in the database according to its sensitivity level (e.g., personal, financial, operational). Evaluate whether the AI would need direct access for specific use cases such as query optimization or data-driven feature suggestions.

2. Consider Security Measures

Evaluate the existing security measures in place to protect the database. Are there robust mechanisms like row-level permissions, encryption at rest and in transit, and logging? Would these be sufficient to mitigate risks if an AI tool had direct access?

3. Evaluate AI Tool Maturity and Reliability

Examine the maturity of the AI tool in question. Is it a mature product with a proven track record for reliability and security? Consider its integration capabilities and whether there are any known vulnerabilities or misuse incidents.

4. Define Access Controls and Auditing

Propose granular access controls, such as role-based access control (RBAC) where the AI tool is granted specific permissions only when necessary. Implement a logging mechanism to monitor AI actions and ensure that any unauthorized access attempts can be detected and acted upon quickly.

5. Test and Validate

Conduct thorough testing in a sandbox environment before deploying the solution. Ensure that the AI tool behaves as expected without compromising data security. This might involve setting up mock databases with sample data to simulate real-world scenarios.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, granting direct database access to an AI tool like Copilot/Cursor involves a careful balance between leveraging advanced coding assistance and protecting sensitive information. The key is to assess the specific use cases, implement robust security measures, and continuously monitor for any anomalies.

#### Common mistakes
include:

Overestimating the maturity of the AI tool and underestimating potential risks.
Failing to establish granular access controls and audit trails.
Ignoring the importance of testing in a controlled environment before full deployment.

#### Follow-ups
might explore how you would implement these security measures, or discuss alternative solutions like using APIs for database interactions with proper authentication and authorization checks.

**Comparison:**

```
User Role	Read Permissions	Write Permissions
admin	*	*
developer	selected_tables	none
guest	none	none
```

**Code:**

```python
# Example of granular access control implementation
def grant_db_access(user_role):
    if user_role == "admin":
        return {"read": "*", "write": "*"}
    elif user_role == "developer":
        return {"read": "selected_tables", "write": "none"}
```

**Example input / output:**

```text
Input: User role = developer, requested access to database tables.
Output: Access granted with read permissions on selected_tables and no write permissions.
```

**Data:**

```yaml
# Example of data classification
data_classification:
  personal: ["customer_name", "email"]
  financial: ["transaction_id", "amount"]
  operational: ["log_entries"]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Role
6. Is Admin?
7. Grant Full Access
8. Is Developer?
9. Grant Read on Selected Tables,No Write
10. Deny Access

