---
type: "practice"
difficulty: "Expert"
---

# You need OIDC-based keyless auth from your pipeline to a cloud provider; what's the trust chain, what could a fork PR abuse, and how do you scope the role to prevent lateral movement?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Secrets in Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure secure access in an OIDC-based keyless authentication setup within your CI/CD pipeline, we need to establish a robust trust chain that mitigates risks such as fork PR abuse and ensures proper role scoping to prevent lateral movement. Here’s how we can achieve this:

TL;DR

The core idea is to set up a secure CI/CD pipeline using OIDC for authentication, ensuring the least privilege principle through role-based access control (RBAC) and leveraging service accounts with limited permissions. We'll also implement strict policies on branch protection rules and continuous monitoring.

#### Triaging the Question

This question probes your understanding of secure CI/CD practices, specifically focusing on OIDC-based keyless authentication in a cloud environment. The key aspects include:

Trust Chain: Ensuring that only authorized entities can access resources.
Fork PR Abuse: Preventing unauthorized access through code forking and pull requests (PRs).
Role Scoping: Limiting the permissions of service accounts to reduce the risk of lateral movement.

#### First-Principles Solution

OIDC Authentication Setup:

Use OpenID Connect (OIDC) from your CI/CD pipeline to authenticate against a cloud provider's identity and access management (IAM) system.
The OIDC flow should include steps for token acquisition, validation, and session management.

Service Accounts with Limited Permissions:

Create service accounts in the cloud provider’s IAM system that have minimal permissions required by your CI/CD pipeline.
Use role-based access control (RBAC) to assign roles based on specific tasks within the pipeline, ensuring that each service account only has the necessary permissions.

Branch Protection Rules:

Implement branch protection rules in version control systems (e.g., GitHub, GitLab) to prevent unauthorized code pushes.
For example, require pull approvals or enforce mandatory status checks for certain branches.

Continuous Monitoring and Auditing:

Monitor access patterns and audit logs to detect any suspicious activities.
Set up alerts for unusual access attempts or policy violations.

Fork PR Abuse Mitigation:

Use fork protection policies in version control systems to restrict code forking.
Ensure that only authorized repositories can be cloned or forked, thereby reducing the risk of unauthorized access through PRs.

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: The key is establishing a secure trust chain with OIDC-based authentication, leveraging RBAC for role scoping, implementing strict branch protection rules, and continuously monitoring activities.

#### Common Mistakes
Overly permissive roles assigned to service accounts.
Failing to monitor and audit access patterns.
Not enforcing strict policies on branch protection rules.

#### Example
Workflow

Let’s walk through a simple example of how this might look in practice:

** OIDC Token Request from CI/CD Pipeline**:

PLAINTEXT
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

client_id=ci_cd_pipeline&client_secret=secret&grant_type=client_credentials

** OIDC Token Validation in Cloud Provider**:

PLAINTEXT
POST /validate-token HTTP/1.1
Host: iam.example.com
Authorization: Bearer <token>
Content-Type: application/json

{
  "sub": "ci_cd_pipeline",
  "roles": ["read_only"]
}

Service Account Access:

PLAINTEXT
GET /api/resource HTTP/1.1
Host: api.example.com
Authorization: Bearer <token>
X-Role: read_only
Common Follow-Ups

The interviewer might probe deeper into specific areas such as:

Detailed RBAC implementation.
Branch protection rules in practice.
Monitoring and alerting mechanisms.

By addressing these areas comprehensively, you can ensure a robust security posture for your CI/CD pipeline.

**Comparison:**

```
Action	Service Account Role
Read-only access to API resources.	read_only
Access to version control repositories.	read_write
```

**Code:**

```plaintext
// Example OIDC Token Request from CI/CD Pipeline
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

client_id=ci_cd_pipeline&client_secret=secret&grant_type=client_credentials

// Example OIDC Token Validation in Cloud Provider
POST /validate-token HTTP/1.1
Host: iam.example.com
Authorization: Bearer <token>
Content-Type: application/json

{
  "sub": "ci_cd_pipeline",
  "roles": ["read_only"]
}
```

**Example input / output:**

```text
Input:
- OIDC token request from CI/CD pipeline to cloud provider IAM.
Output:
- Validated OIDC token with assigned roles for specific actions.

Request (CI/CD Pipeline):
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

client_id=ci_cd_pipeline&client_secret=secret&grant_type=client_credentials

Response (IAM System):
POST /validate-token HTTP/1.1
Host: iam.example.com
Authorization: Bearer <token>
Content-Type: application/json

{
  "sub": "ci_cd_pipeline",
  "roles": ["read_only"]
}
```

**Data:**

```json
// Example OIDC Token Response
{
  "access_token": "<token>",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "read_only"
}
```

**Flow:**

1. CI/CD Pipeline
2. OIDC Token Request
3. IAM Validation
4. Service Account with LimitedPermissions
5. API Access
6. Monitoring and Auditing

