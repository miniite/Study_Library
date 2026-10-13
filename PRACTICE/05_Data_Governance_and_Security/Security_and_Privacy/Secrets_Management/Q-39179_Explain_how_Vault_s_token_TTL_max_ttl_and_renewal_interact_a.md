---
type: "practice"
difficulty: "Intermediate"
---

# Explain how Vault's token TTL, max_ttl, and renewal interact, and what happens when a token hits max_ttl.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vault Outage Resilience](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Outage_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Vault is a powerful secrets management system that helps organizations securely store and manage sensitive information like passwords, API keys, and encryption keys. In this explanation, we will focus on how Vault's token TTL, max_ttl, and token renewal interact to ensure the security and availability of your tokens.

#### Triage
The question asks us to understand how Vault handles token lifetimes and renewals, specifically focusing on the token TTL (Time To Live) and max_ttl. This is essential for ensuring that secrets are securely managed and not exposed due to expired tokens. We need to explain the behavior when a token hits its max_ttl, including what happens during normal operation and in case of failure.

#### First-principles solution
In Vault, a token has two time-related properties:

Token TTL (Time To Live): This is the initial duration for which the token can be used before it expires.
Max TTL: This represents the maximum lifetime that can ever be assigned to the token, even if the token is renewed.

When a token is created or renewed, both token TTL and max_ttl are set. The behavior of tokens when they hit their limits is as follows:

Normal Operation: When a token's token TTL expires, it becomes invalid. However, before expiring, Vault allows the token to be renewed automatically (if enabled) or manually.
Renewal: If automatic renewal is configured and the token’s token TTL is about to expire, Vault will attempt to renew the token by extending its lifetime up to max_ttl. The actual time extension can be less than max_ttl, depending on the policy.

When a token hits max_ttl:

Token Expiry: If a renewal request is made but fails (e.g., due to network issues, policy constraints), the token will expire after its current token TTL duration.
No Renewal Attempts: If automatic renewal is not configured or if the policies prevent renewals, once the max_ttl is reached, the token cannot be extended further and will expire.

#### Closing summary
Understanding these interactions helps in designing robust secrets management strategies. Tokens are automatically renewed up to their max_ttl, but this can fail due to various reasons such as policy constraints or network issues. This mechanism ensures that tokens remain active until their maximum allowed lifetime is reached, after which they must be manually renewed if necessary.

#### Common mistakes
and follow-ups:

Misconfigured Policies: Not setting appropriate policies for automatic renewal or max TTL can lead to token expiration.
Network Issues: If a token cannot renew due to network issues, it may expire prematurely. Monitoring and handling such scenarios are crucial.
Token Management: Implementing strategies like token rotation and key management best practices can mitigate risks associated with expired tokens.

**Comparison:**

```
Time	Token Status
0s	Created
3600s	Renewed, valid for 1 hour
5800s	Renewal attempt failed; token expires after 20 seconds of remaining TTL.
```

**Code:**

```go
// Example Vault token policy for automatic renewal and max TTL
path "secrets/*" {
  capabilities = ["create", "read"]
}

token_type = "jwt"
ttl = "3600s" # Initial TTL of the token, set to 1 hour in this example.
max_ttl = "7200s" # Maximum lifetime of the token, set to 2 hours in this example.
```

**Example input / output:**

```text
**Input (Token Creation):**
- `ttl = 3600s` (1 hour)
- `max_ttl = 7200s` (2 hours)

**Output (Normal Operation):**
- Token is created with a TTL of 1 hour.
- Token can be renewed up to its max TTL of 2 hours.

**Input (Token Renewal Failure):**
- Attempt renewal after 5800 seconds (just before `max_ttl`).

**Output (Failure):**
- Token expires after the remaining time in `token TTL`.

**Input (Network Issue During Renewal):**
- Network issues prevent token renewal at 5800 seconds.

**Output (Failure due to Network Issues):**
- Token expires after the remaining time in `token TTL`.
```

**Data:**

```json
{
  "token": {
    "ttl": "3600s",
    "max_ttl": "7200s"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Token Creation
6. TOKEN_TTL
7. Is Token TTL Exceeded?
8. Renewal Attempt
9. Renewal Successful?
10. Token Expired
11. TOKEN_TTL Renewed tomax_ttl
12. Token Still Valid

