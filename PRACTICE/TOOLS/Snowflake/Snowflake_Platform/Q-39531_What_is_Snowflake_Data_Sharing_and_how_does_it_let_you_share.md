---
type: "practice"
difficulty: "Intermediate"
---

# What is Snowflake Data Sharing and how does it let you share data without copying it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Row Access Policies](<../../../../TOOLS/Snowflake/Snowflake_Platform/Row_Access_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Snowflake Data Sharing allows users to securely share access to their data in a governed manner without physically moving or copying the data, leveraging Snowflake's multi-tenant architecture and role-based access control.

Snowflake Data Sharing is a powerful feature that enables organizations to share data across different teams, departments, or even third-party partners while maintaining security and governance. The core concept revolves around creating a secure sharing relationship between multiple Snowflake accounts without copying the actual data files. This setup ensures that only authorized users can access the shared datasets.

To understand how Data Sharing works, let's break down its key components and walk through an example:

Account-Level Security: Each Snowflake account operates in a multi-tenant environment where each tenant (account) is isolated from others. However, this isolation does not mean that data cannot be shared securely.

Sharing Relationship Creation: To share data, you establish a sharing relationship between the source and target accounts. This can be done via an API or through the Snowflake web interface.

Granular Access Control: Once the sharing relationship is set up, users in the target account are granted access to specific datasets within the shared catalog. The level of access (read-only or read-write) can be finely tuned based on role-based permissions.

Data Location and Performance Optimization: Snowflake does not physically copy data during a share; instead, it uses logical references to the source dataset. This ensures that only metadata is transferred, which speeds up sharing processes significantly.

Let's go through an example step-by-step:

Step 1: Source Account Setup

User in the source account creates and loads a table named SalesData.
They decide to share this data with another team or a partner account.

Step 2: Target Account Setup

The target user logs into their Snowflake account.
They request access from the source account, specifying which datasets they need.

Step 3: Sharing Relationship Configuration

In the source account, you use the GRANT command to create a sharing relationship:

SQL
GRANT USAGE ON DATABASE sales_database TO ROLE target_account_role;

This grants the role access to the shared database.

In the target account, you accept the share using the SHARE command:

SQL
ACCEPT SHARE 'source_account_name:sales_database';

Step 4: Access and Usage

The user in the target account can now query the shared table without any physical data transfer.
Performance is optimized as only metadata is moved, and the actual data remains in the source account.

By setting up Data Sharing, organizations can benefit from reduced storage costs, faster data sharing processes, and enhanced security through fine-grained access control. This approach ensures that sensitive data is not exposed to unnecessary parties while still enabling collaboration and analytics across teams.

#### Common mistakes
include misconfiguring the sharing relationships, failing to revoke permissions when no longer needed, or misunderstanding the implications of full vs. read-only access.

