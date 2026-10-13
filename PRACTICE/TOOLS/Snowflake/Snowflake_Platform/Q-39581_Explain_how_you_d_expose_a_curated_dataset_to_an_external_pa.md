---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd expose a curated dataset to an external partner via a Reader Account or a Data Share, and the trade-offs of each.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Row Access Policies](<../../../../TOOLS/Snowflake/Snowflake_Platform/Row_Access_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To expose a curated dataset to an external partner in Snowflake, two primary methods can be used: creating a Reader Account or leveraging Data Shares. Each method has its own set of trade-offs and considerations that must be carefully evaluated based on the specific requirements and constraints of the project.

#### Triage

The core question revolves around understanding how to securely expose data from one organization (the owner) to another (the external partner) in Snowflake. The interviewer is probing your knowledge of Snowflake's security models, data sharing mechanisms, and the associated costs, complexity, and potential risks involved.

#### Assumptions
The dataset is curated and requires control over who can access it.
Security, compliance, and data privacy are paramount concerns.

#### Performance considerations
for both the owner and the partner are important.

#### Solution
Reader Account

A Reader Account in Snowflake allows you to create a separate account that has read-only access to your existing database objects or shares data through a Data Share. Here’s how it works:

Creating the Reader Account:

Log in to the Snowflake web interface as an administrator.
Navigate to the "Account Settings" and then "User Management."
Create a new user with a distinct username and password.
Assign this user to a role that has read-only access, such as READER_ROLE.

Sharing Data:

Go to the "Data Sharing" section in the Snowflake web interface.
Click on "Create Share."
Define the share by specifying the sources (databases, schemas, tables) and roles that you want to include in the share.
Assign the Reader Account as a participant in this Data Share.

Managing Access:

Regularly review and update access permissions for the Reader Account based on ongoing needs.
Implement fine-grained control over data access using row-level security (RLS) if needed.

#### Trade-Offs
of Reader Accounts
Cost: The primary cost is the additional Snowflake account that you need to manage, which comes at a monthly fee.
Complexity: Managing multiple accounts requires careful planning and administration, especially when it comes to access revocation and user management.
Security Compliance: Although Reader Accounts provide read-only access, ensuring compliance with data privacy regulations like GDPR or CCPA can be complex if the external partner needs to handle sensitive data.
Data Share

A Data Share in Snowflake allows you to share your existing databases and schemas with other organizations without creating a new account. This method is more straightforward but comes with its own set of considerations:

Creating the Data Share:

Log in as an administrator.
Go to "Data Sharing" -> "Create Share."
Define the data sources (databases, schemas) that you want to share.
Add the external partner’s account as a participant in this Data Share.

Participation and Access Control:

The external partner logs into their own Snowflake account and can see the shared data under a separate namespace (_SNF_SHARES).
Access permissions are managed within the partner's own Snowflake account, providing them with read-only or restricted access as needed.

Managing Access:

Regularly review and update share settings to reflect any changes in your data or compliance requirements.
Use RLS to apply additional security measures if necessary.

#### Trade-Offs
of Data Shares
Simplicity: Data Shares are easier to set up and manage compared to Reader Accounts since no new accounts need to be created.
Cost Savings: No additional Snowflake account costs, which can save money in the long run.
Security Compliance: While data remains within the owner’s environment, ensuring compliance might require more rigorous internal controls.

#### Closing Summary

Both methods—Reader Account and Data Share—have their own advantages and trade-offs that need to be carefully considered based on specific use cases. Reader Accounts offer more granular control but come with higher costs and complexity, while Data Shares provide simplicity and cost savings at the expense of slightly less fine-grained access management.

#### Common Mistakes/Follow-Ups
Misunderstanding Fine-Grained Controls: Failing to implement row-level security or schema-level restrictions can lead to unauthorized data access.
Neglecting Compliance Requirements: Ensuring that both methods comply with relevant regulations and standards is crucial, especially for handling sensitive data.
Overlooking Performance Impact: Sharing large datasets can impact performance, so it’s important to monitor and optimize accordingly.

**Comparison:**

```
Role	Cost	Complexity	Security Compliance
Reader Account	Additional account	Higher	Moderate
Data Share	No additional costs	Lower	Lower
```

**Code:**

```sql
-- Example SQL to create a Data Share
CREATE SHARE my_data_share;
GRANT USAGE ON DATABASE my_database TO SHARE my_data_share;

-- Example SQL to add an external partner account as a participant in the share
ALTER SHARE my_data_share ADD ACCOUNT 'partner_account';
```

**Example input / output:**

```text
Input:
- Curated dataset in Snowflake owned by the organization.
- External partner who needs read-only access to this data.

Output:
- A Reader Account created for the external partner with appropriate roles and permissions.
- A Data Share configured to include the owner's databases and schemas, shared with the external partner's account.

Dry-run:
1. Log in as an admin -> Create a new Reader Account -> Assign `READER_ROLE`.
2. Go to "Data Sharing" -> Create Share -> Define sources -> Add participant (external partner).
3. Review access settings and apply RLS if necessary.
```

**Data:**

```json
{
  "share_name": "my_data_share",
  "databases": ["my_database"],
  "schemas": ["public"]
}
```

**Flow:**

1. :
2. Create Reader Account
3. Assign Role
4. READER_ROLE
5. Share Database/Schemas
6. Configure Access Control
7. Monitor and Optimize
8. Create Data Share
9. Add Participant
10. Partner Account
11. Grant Usage/Access
12. Manage Access Settings
13. Apply RLS if Necessary

