---
type: "practice"
difficulty: "Basic"
---

# What are the default system-defined roles such as ACCOUNTADMIN, SYSADMIN, and PUBLIC?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Row Access Policies](<../../../../TOOLS/Snowflake/Snowflake_Platform/Row_Access_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Default system-defined roles in Snowflake serve specific purposes within the data warehousing platform, much like how different roles in a company might have distinct responsibilities. For a Data Engineer, understanding these roles is crucial to managing access and permissions effectively.

In summary, ACCOUNTADMIN has full control over the entire account, SYSADMIN manages infrastructure aspects, and PUBLIC provides public access rights.

To delve into each of these roles:

ACCOUNTADMIN: This role has complete control over the Snowflake account, including creating, modifying, or deleting other users, accounts, and organizations.
SYSADMIN: The SYSADMIN role is responsible for managing infrastructure settings such as clusters, network policies, and security settings.
PUBLIC: The PUBLIC role allows any user to interact with tables shared using the SHARE command but does not provide access to any other objects or data.

These roles are built-in and cannot be modified, ensuring they always have a consistent set of permissions.

**Code:**

```sql
-- Example SQL commands to demonstrate role management (not real, for illustration)
GRANT ACCOUNTADMIN ON CLOUD ACCOUNT TO USER 'admin_user';
REVOKE SYSADMIN FROM ROLE 'sys_admin_role';
```

