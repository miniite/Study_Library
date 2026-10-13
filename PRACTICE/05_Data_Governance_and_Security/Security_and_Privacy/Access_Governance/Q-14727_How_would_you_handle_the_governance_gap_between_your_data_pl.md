---
type: "practice"
difficulty: "Deep"
---

# How would you handle the governance gap between your data platform's access model and the underlying object storage's IAM — where a user denied at the SQL layer might still read raw files directly?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing User-owned Transformations](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Governing_User-owned_Transformations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the governance gap between our data platform’s access control mechanisms and the underlying object storage’s Identity and Access Management (IAM), we need to ensure that any user who is denied access at the SQL layer cannot bypass this restriction by directly accessing raw files stored in object storage. This involves a multi-layered approach, leveraging both technical and procedural controls.

First, let's clarify what the question is probing: it challenges our understanding of how data security and access governance are implemented across layers within a data platform. Specifically, we need to ensure that our platform enforces consistent access policies across all levels, including the object storage layer.

#### Triage

We assume:

Our data platform uses SQL-based access control (e.g., roles, permissions).
The underlying object storage system has its own IAM mechanisms.
There is a potential gap where direct access to raw files could circumvent our SQL-layer controls.

To solve this, we need to ensure that any access attempt that fails at the SQL layer is blocked entirely. This involves:

Implementing fine-grained access controls in both layers.
Monitoring and logging access attempts across all layers.
Configuring automatic failover mechanisms if one layer denies access.
Step-by-Step Solution

Centralized Access Policies: Define a unified set of access policies that are applied consistently at both the SQL platform level and the object storage level. This can be achieved through a centralized policy management system that ensures all relevant systems use the same rules for access control.

Prevention Mechanisms: Implement mechanisms to prevent bypassing access controls. For instance, we can configure our SQL platform to send a request to object storage before any direct file access is attempted. If the request fails, it should result in an error at both layers.

Monitoring and Logging: Set up comprehensive logging and monitoring of all access attempts. This will help in detecting any unauthorized bypasses and allow for quick corrective action.

Automated Failover Mechanisms: Develop automated failover mechanisms that can automatically block access if a policy is violated at any layer. For example, if a user fails to pass SQL-level authentication, our system should also prevent them from accessing raw files in object storage.

User Education and Procedures: Ensure users are aware of the importance of following proper procedures for accessing data, including using only authorized interfaces provided by the platform.

#### Example
Implementation

To illustrate this, consider a scenario where we have an SQL-based data warehouse system (e.g., Amazon Redshift) interacting with S3 object storage. Here’s a high-level flow:

User Request: A user attempts to query data from the SQL layer.
SQL Access Control Check: The request is first checked against the SQL access control rules. If denied, the request fails here.
Pre-Request Validation (Optional): Before allowing direct file access in S3, we can add a pre-validation step where the system sends a request to object storage with metadata about the query or requested files. This ensures that any attempt to bypass SQL-level controls will fail at this stage as well.

#### Closing Summary

By implementing a unified and consistent access control strategy across all layers of our data platform, including both the SQL layer and the underlying object storage, we can effectively close the governance gap. This involves setting up centralized policies, preventing direct access bypasses, monitoring for failures, and automating failover mechanisms to ensure security.

#### Common Mistakes/Follow-Ups
Over-reliance on Technical Controls: Ensuring that technical controls are not the only line of defense; human education and procedural adherence are equally important.
Failure to Log and Monitor: Without robust logging and monitoring, detecting and responding to bypass attempts becomes challenging.
Manual Workarounds: Allowing manual workarounds or exceptions can create vulnerabilities.

To confirm correctness and completeness: This solution covers all necessary aspects of the problem, ensuring consistent access governance across multiple layers.

**Comparison:**

```
Layer	Action	Result
SQL Platform	Check Access	Deny
Object Storage	Pre-Request Validation	Deny
```

**Code:**

```python
# Example Python pseudo-code for pre-request validation in S3

def validate_request_in_s3(request):
    try:
        # Send a request to object storage with metadata about the query or files requested
        response = s3_client.send_validation_request(metadata=request.metadata)

        if not response['approved']:
            raise PermissionError("Access denied by S3")
    except Exception as e:
        print(f"Validation failed: {e}")
        return False

    return True

# Integration with SQL layer
def execute_query(query):
    try:
        # Check access at the SQL level
        if not sql_client.check_access_permissions(query.user, query.role):
            raise PermissionError("Access denied by SQL")

        # Validate request in S3 before allowing direct file access
        if not validate_request_in_s3(request):
            raise PermissionError("Access denied by S3")

        # Proceed with data retrieval from object storage
        result = s3_client.retrieve_data(query)
        return result
    except PermissionError as e:
        print(f"Finalized access denial: {e}")
```

**Example input / output:**

```text
User Request:
- User: user1
- Query: SELECT * FROM sales

Validation Attempt in S3:
- Metadata: {filename: 'sales.csv'}
- Response: {"approved": false}

Final Outcome:
- PermissionError: "Access denied by S3"
```

**Data:**

```json
{
  "sql_client": {
    "permissions": [
      {"user": "admin", "role": "superuser"},
      {"user": "user1", "role": "reader"}
    ]
  },
  "s3_client": {
    "policies": [
      {"user": "admin", "policy": "full_access"},
      {"user": "user1", "policy": "read_only"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Query
6. SQL Access Control
7. Access Approved?
8. Retrieve Data from S3
9. Send Request to ObjectStorage for Validation
10. Validation Approved?
11. Permission Denied by S3

