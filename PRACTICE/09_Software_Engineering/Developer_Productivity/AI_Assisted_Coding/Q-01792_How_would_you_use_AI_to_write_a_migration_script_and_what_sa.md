---
type: "practice"
difficulty: "Deep"
---

# How would you use AI to write a migration script and what safety checks are non-negotiable?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trust Boundaries for AI Generated Code](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Trust_Boundaries_for_AI_Generated_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To develop an AI-assisted coding system that writes a migration script, we need to leverage the strengths of modern AI while ensuring robust safety mechanisms. The core idea is to use natural language inputs and code generation capabilities provided by AI tools like Copilot or similar services. These tools can analyze the current database schema, understand the intended changes, and generate the necessary migration scripts with minimal human oversight.

The question primarily probes your understanding of how to integrate AI-assisted coding in a structured manner while maintaining high standards of safety. Key assumptions include having access to an API for AI code generation, a clear understanding of the application's database structure, and familiarity with best practices in database migrations.

Let's break this down from first principles:

Step 1: Define the Problem

We need to create a migration script that transforms one version of a database schema into another. This involves generating SQL statements or migration scripts based on a detailed description provided via natural language input.

Step 2: Interface with AI Tool

AI tools can help by providing code snippets and suggestions. For instance, if we want to add a new column to the users table, the AI might suggest:

SQL
ALTER TABLE users ADD COLUMN is_admin BOOLEAN DEFAULT FALSE;

The challenge here is to ensure that such suggestions are valid, safe, and align with the existing database schema.

Step 3: Safety Checks

Non-negotiable safety checks include:

Schema Validation: Ensure the proposed changes do not violate any constraints or relationships in the existing schema.
Data Consistency: Verify that the suggested migrations will not lead to data loss or corruption. For example, adding a foreign key without ensuring referential integrity would be problematic.
Transaction Management: Use transactions to ensure atomicity; if a migration fails at any point, all changes should roll back cleanly.
Step 4: Human Review

Even with advanced AI, human review is crucial. The generated scripts should undergo thorough testing in a staging environment before deployment to production. This includes:

Running the script against a copy of the live database.
Monitoring for unexpected behavior or errors during execution.
Ensuring that all intended changes have been applied correctly.
Step 5: Documentation and Rollback Plan

Document every migration clearly, including the rationale for each change. Have a rollback plan ready in case something goes wrong. This might involve maintaining separate versioned backups of the database schema and data.

By integrating these steps, we can leverage AI to enhance our productivity while still ensuring that safety is prioritized. The final solution combines the power of AI with rigorous safety measures to produce reliable migration scripts.

#### Closing Summary

In summary, using AI for writing migration scripts requires careful integration with robust safety mechanisms. Key steps include defining the problem clearly, interfacing with an AI tool, implementing non-negotiable safety checks, conducting thorough human review, and maintaining comprehensive documentation and rollback plans. The goal is to balance efficiency gains from automation with the necessity of maintaining high standards in database operations.

#### Common Mistakes/Follow-Ups
Overreliance on AI: Failing to perform necessary manual validation can lead to critical errors.
Lack of Rollback Plan: Without a clear rollback strategy, any issues discovered post-deployment could be difficult to resolve.
Ignoring Schema Validation: Ensuring that proposed changes do not violate existing constraints is crucial for database integrity.

**Comparison:**

```
Migration Step	Description
Add Column	Adds is_admin column with appropriate constraints.
Data Validation	Checks existing rows to ensure no conflicts.
Transaction	Ensures atomicity using database transactions.
```

**Code:**

```python
# Example API interaction with an AI tool
response = ai_tool.generate_script("Add a new column 'is_admin' to the users table")
migration_script = response['script']
print(migration_script)
```

**Example input / output:**

```
Input: "Add a new column 'is_admin' to the users table."
Output:
SQL
Copy
ALTER TABLE users ADD COLUMN is_admin BOOLEAN DEFAULT FALSE;
Dry Run in Staging Environment:
Apply the script.
Monitor for any issues.
Ensure no data loss or corruption.
```

**Data:**

```
users Table Schema:
SQL
Copy
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(100) NOT NULL UNIQUE,
  email VARCHAR(255),
  password_hash BYTEA NOT NULL,
  is_admin BOOLEAN DEFAULT FALSE CHECK (is_admin IN (TRUE, FALSE))
);
```

**Flow:**

1. Yes
2. No
3. Start
4. Is AI Suggested Script Valid?
5. Migrate in Staging
6. Monitor & Validate
7. Migrate to Production
8. Raise Error, Review Suggestion
9. Fix Issue and Retry from B

