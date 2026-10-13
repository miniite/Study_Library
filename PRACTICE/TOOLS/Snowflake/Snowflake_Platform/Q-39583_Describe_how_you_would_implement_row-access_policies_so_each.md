---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you would implement row-access policies so each regional team sees only its own rows, and how you'd test it doesn't leak.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Row Access Policies](<../../../../TOOLS/Snowflake/Snowflake_Platform/Row_Access_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that each regional team in Snowflake can access only their respective data, we need to leverage Snowflake's row-access policies (RAPs) which allow us to implement fine-grained control over who can see what data based on specific conditions. This approach ensures that teams do not have access to sensitive information from other regions.

#### Triaging the Question

The question is essentially about designing a security policy in Snowflake to restrict data visibility based on regional affiliations. The key points are:

Defining Row-Access Policies: Creating RAPs with appropriate conditions.
Ensuring Data Isolation: Ensuring that policies do not inadvertently expose data from other regions.

#### Assumptions
and First Principles

Assume we have a regions table in the database, which contains information about each region's ID and name. We will use this as the basis for our row-access policies. The steps to implement and test these policies are:

Define the regions.
Create RAPs based on the region IDs.
Apply these policies to tables containing sensitive data.
Test the implementation to ensure no data leakage.
Detailed Solution
Step 1: Defining Regions

First, we need a table that defines our regions:

SQL
CREATE TABLE regions (
    region_id INT PRIMARY KEY,
    region_name VARCHAR(255)
);

For simplicity, let's assume it contains the following data:

region_id	region_name
1	US
2	Europe
3	Asia
Step 2: Creating Row-Access Policies

Next, we create RAPs based on the regions table. Each policy will restrict access to data where the region ID matches.

SQL
-- Policy for Region 1 (US)
CREATE ROW ACCESS POLICY us_policy ON sales_data FOR SELECT ASOF 'region_id' = 1;

-- Policy for Region 2 (Europe)
CREATE ROW ACCESS POLICY europe_policy ON sales_data FOR SELECT ASOF 'region_id' = 2;

-- Policy for Region 3 (Asia)
CREATE ROW ACCESS POLICY asia_policy ON sales_data FOR SELECT ASOF 'region_id' = 3;

In the sales_data table, we have sensitive information like customer orders and prices. By applying these policies, only users associated with a specific region will be able to query data that matches their region ID.

Step 3: Applying Policies

We apply these RAPs to ensure that each user can only see data relevant to their assigned region:

SQL
-- Assuming user1 is from US and has access to us_policy
GRANT EXECUTE ON ROW ACCESS POLICY us_policy TO USER user1;

-- Similarly, for other users
Step 4: Testing the Implementation

To test that no data leakage occurs, we can use a few approaches:

Manual Query Tests: Run queries from each region and ensure they only return results related to their own region.
Data Masking: Use Snowflake's masking feature to hide sensitive data in development environments.
Role-Based Testing: Create different roles for users and test if access is correctly restricted.

#### Closing Summary

By leveraging row-access policies, we can effectively control who sees what data in a multi-regional environment. This ensures that each team only has access to their assigned region's information, preventing accidental or unauthorized data exposure.

#### Common Mistakes
and Follow-Ups
Misconfigured Policies: Ensure the RAPs are correctly defined and applied.
Inconsistent Data Distribution: Verify that all relevant regions are represented in your regions table.
Overlapping Regions: Check for any edge cases where multiple policies might overlap, causing unintended access.

