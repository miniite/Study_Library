---
type: "practice"
difficulty: "Expert"
---

# How would you periodically rekey Vault (rotate the master encryption key and unseal shares) with minimal risk to a live cluster?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rotation with Dynamic and Static Secrets](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Rotation_with_Dynamic_and_Static_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the security of our data and maintain high availability in a live Vault cluster, periodic rekeying is essential. This process involves rotating the master encryption key while ensuring minimal disruption to ongoing operations. Let's break down the steps required to achieve this with minimal risk.

Firstly, let’s triage the question: The core challenge here is to rekey the Vault cluster without compromising service availability or data integrity. We need a plan that ensures all unseal shares are updated, and new keys are properly generated and distributed before removing old ones. This involves understanding the Vault architecture, key lifecycle management, and using built-in tools for smooth transitions.

#### Assumptions

The current Vault cluster is live and being used.
There are multiple unseal shares available for redundancy.
We use a multi-unseal configuration to ensure high availability.
We have a backup of all existing keys and data.
Step-by-Step Solution

Preparation Phase:

Ensure you have the latest Vault version installed, which supports rekeying.
Backup all current keys, tokens, and secrets in case something goes wrong.
Verify that your cluster has enough unseal shares to remain operational during the transition.

Rotation Process: a. Update Unseal Shares:

BASH
vault operator rekey --current-unseal-key <CURRENT_UNSEAL_KEY>

This command will initiate the rekeying process and generate new unseal keys.

b. Distribute New Keys: Once the new key is generated, distribute it to all nodes in the cluster.

BASH
vault operator unseal --unseal-key <NEW_UNSEAL_KEY>

Repeat this command until the entire cluster acknowledges the new key.

c. Update Configuration Files: Update any configuration files or scripts that rely on Vault keys to use the new ones.

Verification Phase:

Confirm that all nodes in the cluster can unseal with the new keys.
BASH
vault operator unseal --unseal-key <NEW_UNSEAL_KEY>
Ensure that no old keys are being used by any node or client.

Cleanup Phase:

Once you're confident that all nodes and services are using the new key, start removing old keys.
BASH
vault operator rekey --final-rekey
This command finalizes the rekeying process and removes the current key from use.

Post-Rotation:

Monitor Vault logs for any issues during or after the rekey operation.
Ensure all services that depend on Vault are configured to use the new keys.

#### Closing Summary

By following these steps, you can ensure a smooth transition of Vault’s master encryption key without disrupting service availability. The key is to have redundancy in unseal shares and thorough verification before removing old keys. Always keep backups and monitor systems closely during this process.

#### Common Mistakes/Follow-Ups
Failing to Backup Keys: Always back up the current state, as any failure can lead to data loss or service disruption.
Incorrect Key Distribution: Ensuring all nodes have updated unseal shares is crucial; otherwise, the cluster might become unavailable.
Insufficient Monitoring: Regularly monitor logs and metrics during rekey operations to catch and resolve issues early.

**Code:**

```

vault operator rekey --current-unseal-key <CURRENT_UNSEAL_KEY>

CODE
Copy

```bash
vault operator unseal --unseal-key <NEW_UNSEAL_KEY>
BASH
Copy
vault operator rekey --final-rekey
```

