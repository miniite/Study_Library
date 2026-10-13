---
type: "practice"
difficulty: "Expert"
---

# Your identity provider is migrating from a legacy SAML setup to OIDC. Design the Vault identity/auth transition so 10000 human and machine identities keep working through the cutover with no downtime.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Rotation with Dynamic and Static Secrets](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Rotation_with_Dynamic_and_Static_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The key to a seamless transition from SAML to OIDC in Vault lies in leveraging intermediate authentication methods, such as an adaptive multi-factor authentication (MFA) approach or a hybrid identity model. We will design this by creating a custom role-based access control (RBAC) strategy within Vault that allows for parallel operation of both legacy and new identities during the cutover period.

#### Triaging the Question

The core requirement is to ensure 10,000 human and machine identities can continue working without downtime during the migration from SAML to OIDC. This involves designing an authentication flow that supports both old and new identities in parallel until the full transition is complete. The interviewer is probing our understanding of Vault’s flexibility, our ability to manage identity transitions at scale, and how we handle edge cases.

#### Assumptions
We have a legacy SAML setup running and need to migrate it to OIDC.
There are 10,000 identities that need to be managed during the transition.
The migration process is gradual, allowing for an overlap period where both systems coexist.
We can use Vault’s identity and auth paths for managing these transitions.
Step-by-Step Solution:
Phase 1: Initial Setup

Configure OIDC Authentication Method in Vault:

Use the auth oidc path to set up a new OIDC authentication method.
Configure the OIDC client with appropriate discovery endpoints and audience claims.

Create an Identity Mapping Role:

Define a custom role within Vault that maps legacy SAML identities to new OIDC identities using attributes like email or username.
HCL
path "oidc-identity-mapping" {
  capabilities = ["create", "update"]
  policies = ["oidc-policy"]

  policy 'oidc-policy' <<EOF
  path "secret/data/*" {
    capabilities = ["read", "list"]
  }
  EOF
}

Set Up SAML Authentication Method:

Ensure the auth saml method is properly configured with the necessary metadata.
HCL
path "saml-identity-mapping" {
  capabilities = ["create", "update"]
  policies = ["saml-policy"]

  policy 'saml-policy' <<EOF
  path "secret/data/*" {
    capabilities = ["read", "list"]
  }
  EOF
}
Phase 2: Transition Management

Parallel Operation:

Allow both SAML and OIDC authentication methods to coexist during the transition.
Configure Vault to use a dynamic policy that checks if an incoming request is coming from either authentication method.

Intermediate Authentication Strategy:

Implement a custom script or API endpoint that validates credentials against both legacy and new systems before allowing access.
HCL
path "intermediate-auth" {
  capabilities = ["create", "update"]
  policies = ["intermediate-policy"]

  policy 'intermediate-policy' <<EOF
  path "secret/data/*" {
    capabilities = ["read", "list"]
  }
  EOF
}

Gradual Rollout:

Use a staged rollout approach where you gradually reduce the SAML-based access over time as more users switch to OIDC.
Monitor and log any discrepancies or issues that arise during this phase.
Phase 3: Final Migration

Disable Legacy Authentication Method:

Once all critical systems are running on OIDC, disable the legacy SAML method.
HCL
path "auth/saml" {
  capabilities = ["create", "update"]
  policies = ["saml-policy"]

  deny = true
}

Purge Legacy Credentials:

Clean up any lingering data or credentials related to the legacy system.

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Not Planning for Transition: Failing to plan a phased transition can lead to confusion and downtime.
Lack of Monitoring: Without proper monitoring, issues in either authentication method may go unnoticed until it’s too late.
Over-Complexity: Over-complicating the solution by not leveraging Vault’s built-in features for identity mapping.

#### Follow-Ups
How do you ensure security during this transition?
What is your approach to handling non-human identities like service accounts?
How would you handle any unforeseen issues that arise during the cutover period?

**Comparison:**

```
Identity Type	Transition Path
Human User	oidc-identity-mapping
Machine	saml-identity-mapping
Phase	Action
Initial Setup	Configure OIDC and SAML
Parallel Op.	Validate credentials
Final Migrate	Disable legacy methods, clean up
```

**Code:**

```hcl
# OIDC Configuration
path "oidc-identity-mapping" {
  capabilities = ["create", "update"]
  policies = ["oidc-policy"]

  policy 'oidc-policy' <<EOF
  path "secret/data/*" {
    capabilities = ["read", "list"]
  }
  EOF
}

# SAML Configuration
path "saml-identity-mapping" {
  capabilities = ["create", "update"]
  policies = ["saml-policy"]

  policy 'saml-policy' <<EOF
  path "secret/data/*" {
    capabilities = ["read", "list"]
  }
  EOF
}

# Intermediate Authentication Path
path "intermediate-auth" {
  capabilities = ["create", "update"]
  policies = ["intermediate-policy"]

  policy 'intermediate-policy' <<EOF
  path "secret/data/*" {
    capabilities = ["read", "list"]
  }
  EOF
}
```

**Example input / output:**

```text
Input (OIDC Token):
{
  "sub": "1234567890",
  "email": "user@example.com"
}

Output:
{
  "auth": {
    "client_token": "generated-token",
    "lease_duration": 3600,
    "lease_id": "/oidc-identity-mapping/user@domain.com/1234567890"
  }
}
```

**Data:**

```json
{
  "oidc-client": {
    "client_id": "example-oidc",
    "token_endpoint": "https://oidc.example.com/token"
  },
  "saml-metadata": "<metadata><EntityDescriptor ID='...'></EntityDescriptor></metadata>"
}
```

**Flow:**

1. Yes
2. No
3. User Authenticates
4. Is Legacy?
5. Authenticate with OIDC
6. Authenticate with SAML
7. Migrate to OIDC
8. Migrate to OIDC
9. Disable Legacy Methods

