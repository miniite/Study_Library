---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd prevent 'access accretion' where employees changing roles keep old data grants forever. Design the joiner-mover-leaver automation and how it ties to the data platform.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Access Governance for LLM Pipelines](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address access accretion in a data engineering context, we need to ensure that employee data access permissions are automatically adjusted when they change roles or leave an organization. This requires a well-orchestrated system of automation and governance integrated with the broader data platform.

#### Triage
Key Elements
Current State: Employees might retain excessive or outdated access rights due to role changes.
Goals: Ensure that:
Access is dynamically adjusted based on current roles.
Historical records are maintained for audit purposes.
Automated processes reduce manual errors and ensure compliance.

#### First-Principles Solution

Data Platform Integration:

Integrate with the data platform to retrieve current role mappings.
Leverage an access governance layer that enforces rules based on roles.

Automated Processes:

Joiner, Mover, and Leaver (JML) workflows need to be defined.
Use a change detection mechanism to trigger these processes.

Role Mapping and Access Revocation/Granting:

Maintain an up-to-date mapping of roles and associated data access.
Ensure that permissions are revoked or granted based on the latest role mappings.

Audit Logging and Compliance:

Log all changes for audit purposes.
Ensure compliance with regulatory requirements.
Step-by-Step Solution

Define Role Mappings:

Create a structured role mapping system where each employee has an associated set of roles.

#### Example
user_id -> [role_1, role_2]

Joiner Workflow:

When a new employee joins (e.g., HR record update):
Trigger the JML workflow.
Assign initial access based on their role mappings.

Mover Workflow:

When an employee changes roles within the organization (e.g., through HR records):
Detect the change in roles via a change detection mechanism.
Revoke old permissions and grant new ones as needed.
Update the role mapping system.

Leaver Workflow:

When an employee leaves (e.g., termination or resignation):
Trigger the JML workflow to remove all associated access rights.
Archive historical data access logs for audit purposes.
Update HR records and role mappings accordingly.
Integration with Data Platform

APIs and Services:

Develop APIs to integrate with existing systems (HR, Access Governance).

#### Example
GET /user/roles/{user_id}

Access Control Lists (ACLs):

Use ACLs in the data platform to enforce access based on roles.

#### Example
JSON schema for ACLs:
JSON
{
  "access_control_lists": [
    {
      "user_id": "12345",
      "roles": ["admin", "reader"],
      "data_sets": ["sales_data", "customer_info"]
    }
  ]
}
Automation Framework
JML Workflow Engine:
Implement a workflow engine that handles joiner, mover, and leaver processes.

#### Example
code snippet for triggering workflows:
PYTHON
def handle_user_event(user_id, event_type):
    if event_type == "join":
        assign_initial_access(user_id)
    elif event_type == "move":
        update_access_based_on_new_role(user_id)
    elif event_type == "leave":
        revoke_all_access(user_id)
Audit and Compliance
Audit Logs:
Maintain detailed logs of access changes for audit purposes.

#### Example
log entry:
JSON
{
  "timestamp": "2023-10-01T14:30:00Z",
  "user_id": "12345",
  "old_roles": ["admin"],
  "new_roles": ["reader", "writer"],
  "action": "role_update"
}

#### Common Mistakes
and Follow-Ups
Manual Interventions: Ensure that no manual steps are required for critical processes.
Data Consistency: Regularly validate the consistency of role mappings across systems.
Performance Impact: Monitor the performance impact of automated processes, especially during large-scale changes.

**Comparison:**

| Event Type | Action                 |
|------------|------------------------|
| Join       | Assign initial access  |
| Move       | Update access          |
| Leave      | Revoke all access      |

**Code:**

```python
def handle_user_event(user_id, event_type):
    if event_type == "join":
        assign_initial_access(user_id)
    elif event_type == "move":
        update_access_based_on_new_role(user_id)
    elif event_type == "leave":
        revoke_all_access(user_id)

def assign_initial_access(user_id):
    # Implement logic to assign initial access based on role mappings
    pass

def update_access_based_on_new_role(user_id):
    # Implement logic to update access permissions
    pass

def revoke_all_access(user_id):
    # Implement logic to remove all access rights for the user
    pass
```

**Example input / output:**

```

Example input and output:

Input: User 12345 role update from admin to reader.
Output:
JSON
Copy
{
  "timestamp": "2023-10-01T14:30:00Z",
  "user_id": "12345",
  "old_roles": ["admin"],
  "new_roles": ["reader", "writer"],
  "action": "role_update"
}
```

**Data:**

```json
{
  "access_control_lists": [
    {
      "user_id": "12345",
      "roles": ["admin", "reader"],
      "data_sets": ["sales_data", "customer_info"]
    }
  ]
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Joiner] --> B{Is user joining?};
4. B -- Yes --> C[Assign initial access];
5. B -- No --> D[Mover]{Is role changing?};
6. D -- Yes --> E[Update access based on new role];
7. D -- No --> F[Leaver]{Is user leaving?};
8. F -- Yes --> G[Rename all access];
9. F -- No --> H[Audit logging]{Log changes for audit purposes};

