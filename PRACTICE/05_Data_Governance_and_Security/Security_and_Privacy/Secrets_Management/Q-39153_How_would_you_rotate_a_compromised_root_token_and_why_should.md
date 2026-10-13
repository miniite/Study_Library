---
type: "practice"
difficulty: "Deep"
---

# How would you rotate a compromised root token, and why should the root token normally not exist at all in steady state?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shared Secret Rotation](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Shared_Secret_Rotation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage secrets effectively, especially with a root token, we need to understand the core principles of secrets management and the importance of minimizing risk exposure. A compromised root token can be a significant security breach, so its rotation is critical. The root token should not normally exist in steady-state operations due to the high-risk nature associated with it.

The question asks us to discuss how to rotate a compromised root token and why it shouldn't typically exist. This involves understanding the concept of least privilege, ensuring that root tokens are used only during critical initialization phases, and implementing robust rotation procedures.

**Comparison:**

```
Step	Action
1	Generate a new root token using the existing Vault client.
2	Verify the new root token's validity within the Vault environment.
3	Log the retirement of the old root token.
4	Replace the old token with the new one in all relevant systems and configurations.
```

**Code:**

```python
# Example Python function for rotating a token
def rotate_token(current_token, vault_client):
    new_token = vault_client.generate_new_root_token()
    # Log the old token to be retired
    logging.info(f"Rotating root token. Old token={current_token}")
    # Retire the old token after verifying new one is working
    if vault_client.verify_new_token(new_token):
        return new_token
    else:
        raise ValueError("Failed to rotate token")
```

**Example input / output:**

```text
Input: Current Root Token = "old-root-token", New Root Token = "new-root-token"
Output:
- Old token retired successfully.
- New root token active and verified in Vault.
```

**Data:**

```yaml
# Example Vault configuration for root token rotation
root_token_rotation:
  current_root_token: "旧令牌值"
  new_root_token: "新令牌值"
  verification_status: true
```

**Flow:**

1. No
2. Yes
3. Identify Compromise
4. Is Root Token Compromised?
5. Verify New Token
6. Retire Old Token
7. Generate New Root Token
8. Verify New Token
9. Retire Old Token
10. Replace with New Token inSystems

