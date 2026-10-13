---
type: "practice"
difficulty: "Deep"
---

# Design secrets management + rotation for DB passwords and API keys across a fleet so a rotation never causes an outage. How do apps pick up a rotated secret without a redeploy, and how do you handle overlap windows?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design secrets management and rotation for database passwords and API keys across a fleet of applications, we need to ensure that a rotation does not cause outages or downtime. The key challenge here is ensuring that apps can pick up rotated secrets without requiring redeployment, while also managing the overlap window where both old and new secrets coexist.

Triaging the Problem

The core issue involves two primary aspects:

Secret Rotation: Ensuring seamless rotation of secrets like database passwords or API keys.
Zero-Downtime Deployment: Allowing applications to switch over to the new secret without a full redeployment, ensuring continuity and no service disruption.

#### First-Principles Solution

To address this problem effectively, we can employ a combination of techniques including:

Secrets Management System (SMS): A centralized repository for storing secrets securely.
Stale Secret Detection: Mechanisms to detect when the current secret is about to be rotated.
Versioned Secrets: Storing both old and new versions of secrets to ensure backward compatibility.
Automatic Secret Switching: Tools or services that can automatically switch applications from one version of a secret to another.
Detailed Steps
Step 1: Centralized Secrets Management

Implement a centralized secrets management system (SMS) such as AWS Secrets Manager, Hashicorp Vault, or custom-built systems. This ensures all secrets are stored securely and accessed through well-defined APIs.

Centralized SecretsManagement System
Store Secrets
APIs for Retrieval
Step 2: Versioned Secrets

When rotating a secret, store both the old (current) version and the new version of the secret. This allows applications to switch between them smoothly.

PYTHON
# Example Secret Rotation in SMS
def rotate_secret(secret_name):
    # Fetch current secret from SMS
    current_secret = get_current_secret(secret_name)

    # Rotate secret by updating it in SMS
    update_secret(secret_name, new_value)

    # Mark the old version as "current" and the new one as "upcoming"
    mark_current_as_upcoming(current_secret)
    mark_new_as_current(new_value)

# Example Secret Retrieval Logic
def get_secret(secret_name):
    if is_in_overlap_window():
        return get_current_secret(secret_name)  # Return current version
    else:
        return get_upcoming_secret(secret_name)  # Return upcoming (new) version
Step 3: Automatic Secret Switching

Implement a mechanism to automatically switch secrets without requiring application redeployment. This can be done by using environment variables or configuration files that are dynamically updated.

Application
Retrieve Secrets
Environment Variables / ConfigFiles
Step 4: Stale Secret Detection

Set up monitoring to detect when the current secret is about to be rotated. This ensures that applications can gracefully switch over during the overlap window.

PYTHON
def monitor_secret_rotation(secret_name):
    if is_secret_close_to_expiration(secret_name):
        notify_application(secret_name)
Step 5: Overlap Window Management

During the overlap window, ensure that both old and new secrets are available to applications. This can be managed by setting up a gradual rollout strategy.

Application
Retrieve Secret from SMS
Check Current Version
Use Current Version (if not inoverlap)
Switch to Upcoming Version (ifin overlap)

#### Closing Summary

By implementing a centralized secrets management system, versioned secrets, automatic switching logic, and monitoring for stale secrets, we can ensure that secret rotations do not cause outages. The key is to manage the transition carefully during the overlap window.

#### Common Mistakes/Follow-Ups
Missing Stale Secret Detection: Failing to set up mechanisms to detect when a secret needs rotation.
No Automatic Switching Logic: Relying on manual intervention for switching secrets, which can introduce delays and errors.
Insufficient Monitoring: Not setting up robust monitoring to alert applications during the overlap window.

**Comparison:**

```
Step	Action
1	Centralized Secrets Management System setup
2	Versioned Secret Storage Implementation
3	Automatic Secret Switching Logic
4	Stale Secret Detection Mechanism Setup
5	Overlap Window Management Strategy
```

**Code:**

```python
def rotate_secret(secret_name):
    current_secret = get_current_secret(secret_name)
    update_secret(secret_name, new_value)
    mark_current_as_upcoming(current_secret)
    mark_new_as_current(new_value)

def get_secret(secret_name):
    if is_in_overlap_window():
        return get_current_secret(secret_name)
    else:
        return get_upcoming_secret(secret_name)

def monitor_secret_rotation(secret_name):
    if is_secret_close_to_expiration(secret_name):
        notify_application(secret_name)
```

**Example input / output:**

```text
Input:
- Secret Name: db_password

Output:
- Before Rotation: Current Value = old_password123, Upcoming Value = new_password456
- During Overlap Window: Application retrieves old_password123
- After Rotation: Application switches to new_password456
```

**Data:**

```json
{
  "secret_name": "db_password",
  "current_value": "old_password123",
  "upcoming_value": "new_password456"
}
```

**Flow:**

1. Application
2. Retrieve Secrets
3. Check Current Version
4. Use Current Version (if not inoverlap)
5. Switch to Upcoming Version (ifin overlap)

