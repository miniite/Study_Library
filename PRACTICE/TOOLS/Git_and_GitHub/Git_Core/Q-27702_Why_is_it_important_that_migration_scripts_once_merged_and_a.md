---
type: "practice"
difficulty: "Intermediate"
---

# Why is it important that migration scripts, once merged and applied, are never edited in place but instead corrected by a new migration?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Versioning SQL and Migrations](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_SQL_and_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why migration scripts should not be edited in place after they have been merged and applied, we need to delve into the principles of version control and database management. This practice ensures consistency, traceability, and reliability in our database schema changes.

TL;DR

Migration scripts should never be edited in place once they are merged and applied because doing so disrupts the version history and makes it difficult to track changes. Instead, corrections should be made by creating new migration files that accurately reflect the desired state of the database schema.

#### Triaging the Question

The question is essentially asking about best practices for managing database migrations within a version control system like Git using GitHub. The key aspects being probed are:

Version Control: How changes to code and configurations should be managed in a systematic way.
Database Schema Management: Ensuring that schema changes are tracked and can be easily rolled back or reproduced.

#### Assumptions
We are working with a database management system (DBMS) that supports migrations, such as PostgreSQL, MySQL, or SQLite with tools like Flyway or Liquibase.
The team is using Git and GitHub for version control of their codebase, including migration scripts.

#### First Principles Solution
Immutable History: Each migration script represents a specific change to the database schema at a particular point in time. Edits made to an already applied migration can alter its meaning or history, making it difficult to track and revert changes accurately.
Traceability and Rollback: If we edit an existing migration, it becomes unclear which version of the migration was actually executed during deployment. This lack of clarity makes rollback processes more complex and error-prone.
Reproducibility: Future developers or deployments might expect a specific set of migrations to be run in a given order. Editing old migrations could break this expected sequence.

To maintain integrity, we should always:

Create new migration scripts for changes.
Reference the correct version of the migration when applying it.
Ensure that the database schema is consistent with the latest migration history.
Full Solution

Create New Migration Scripts: When a requirement change or bug fix necessitates altering the database schema, start by creating a new migration script. This ensures that the old script remains immutable and its history is preserved.

Apply New Migration: Run the new migration script to update the schema. Ensure it correctly reflects the intended changes without modifying existing migrations.

Review and Commit: Review the new migration script, commit it to Git with a descriptive message explaining the change, and push it to GitHub.

Automated Testing: Validate that the new migration works as expected through automated tests before merging it into the main branch.

Incremental Backward Compatibility: Ensure that new migrations can be run against older database states without causing issues. This involves carefully designing migrations with backward compatibility in mind, often using conditional logic or fallback mechanisms.

#### Closing Summary

By following this approach of never editing applied migration scripts, we ensure a clean and maintainable version history. New changes are documented explicitly, making it easier to understand the evolution of the database schema over time. Additionally, this practice facilitates better collaboration among team members and simplifies troubleshooting during deployments or rollbacks.

#### Common Mistakes/Follow-Ups

Mistake: Editing existing migration scripts in place.

#### Follow-Up
Always create new migration files for changes to maintain a clear and traceable history.

Mistake: Not committing or pushing changes to version control.

#### Follow-Up
Ensure all migrations are version-controlled and follow standard commit messages.

**Comparison:**

```
Migration File	Description
v1	Initial creation of the users table.
v2	Adding the phone_number field
```

**Example input / output:**

```text
Input:
- Existing migration script (version 1)
- New requirement: Add 'phone_number' field

Output:
- New migration file created for adding 'phone_number'
- Original migration file remains unchanged
- Commit message: "Add phone number column to users table"
```

**Flow:**

1. Requirement Change
2. Create New Migration Script
3. Apply New Migration
4. Commit and Push to GitHub
5. Run Automated Tests

