---
type: "practice"
difficulty: "Deep"
---

# When you REVOKE a privilege that was granted WITH GRANT OPTION and re-granted by the grantee, what happens to the downstream grants?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Privileges and Roles](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Privileges_and_Roles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you revoke a privilege that was granted with the GRANT OPTION clause, and that privilege has been further granted by the recipient (grantee), the key point is understanding how this affects the downstream grants. The core of the problem lies in the hierarchical nature of privileges in SQL databases.

To break down the question:

Understanding the Initial Grant: When a user (the grantor) grants a privilege with GRANT OPTION, they allow the recipient to further grant that same privilege to others.
Revoke Action: Revoking a privilege that was granted with GRANT OPTION can have cascading effects, as it not only removes the direct privilege but also affects any downstream grants.

Let's reason through this step by step:

Initial Setup:

User A (grantor) grants SELECT privilege on table T to User B (grantee) with the GRANT OPTION.
User B can now grant SELECT privilege on table T to other users, such as User C.

Revoking from Grantor:

If User A revokes the SELECT privilege on table T granted to User B with GRANT OPTION, it directly removes that specific privilege for User B.
However, since the privilege was granted with GRANT OPTION, User B can still grant this privilege further down the line.

Downstream Grants:

If User B has already granted SELECT privilege on table T to User C (or any other user), these downstream grants are not immediately affected by A's revocation.
The key point is that once a privilege with GRANT OPTION is revoked from the grantor, it does not automatically propagate up and revoke all downstream grants.

Resolution:

To fully remove the privilege from all users who have received it, you need to perform two actions:
Revoke the privilege directly from User B (the initial grantee).
If necessary, manually check and revoke any additional downstream grants that User B might have made.

In summary, revoking a GRANT OPTION-based privilege only removes it for the direct recipient and does not immediately affect any downstream grants. You need to ensure all downstream recipients of this privilege are also revoked directly or by checking their permissions.

**Comparison:**

| Grantor | Grantee Initial | Downstream Grantee |
|---------|----------------|--------------------|
| USER_A  | USER_B          | USER_C             |

**Code:**

```sql
-- Example SQL commands

-- Grant SELECT on table T with GRANT OPTION to User B
GRANT SELECT ON table T TO USER B WITH GRANT OPTION;

-- User B grants SELECT on table T to User C
GRANT SELECT ON table T TO USER C;

-- User A revokes the privilege from User B (direct recipient)
REVOKE SELECT ON table T FROM USER B;

-- To fully remove it, you may need to manually check and revoke any downstream grants from User B
```

**Example input / output:**

```

Initial Setup:

SQL
Copy
GRANT SELECT ON table T TO USER B WITH GRANT OPTION;

Downstream Grant:

SQL
Copy
GRANT SELECT ON table T TO USER C;

Revocation Action (Directly from Grantor):

SQL
Copy
REVOKE SELECT ON table T FROM USER B;

Check and Revoke Downstream Grants if necessary:

SQL
Copy
-- Check User_B's permissions
SHOW GRANTS FOR USER C;

-- If found, revoke the grant from User_C
REVOKE SELECT ON table T FROM USER C;
```

**Data:**

```json
{
  "grantor": "USER_A",
  "grantee_initial": "USER_B",
  "downstream_grantee": "USER_C"
}
```

**Flow:**

1. Grant SELECT on table T toUser B with GRANT OPTION
2. User B grants SELECT on tableT to User C
3. Revoke SELECT on table T fromUser B directly by grantor (A)
4. Check and manually revokeany downstream grants ifnecessary

