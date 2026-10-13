---
type: "practice"
difficulty: "Expert"
---

# You want every deployed schema state to be provably traceable to a reviewed Git commit, but DBAs occasionally apply emergency changes. Design a break-glass path that still leaves an immutable audit trail in Git and a reconciliation migration afterward.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Branch Protection Design](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branch_Protection_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure every deployed schema state is provably traceable to a reviewed Git commit while allowing emergency changes, we need to design a process that includes both controlled updates through review and commit cycles, as well as an immediate rollback or reconciliation path for emergencies. The key is maintaining immutability in the primary codebase while providing flexibility during critical situations.

#### Triaging the Question

The question asks us to balance two critical needs: 1) ensuring every schema change is reviewed and committed to a Git repository to maintain audit trails, and 2) allowing emergency changes that bypass normal review processes but still must be audited and reconciled post-hoc. We need to design a system where both regular and emergency changes are traceable and reconcilable.

#### Assumptions
Review Process: A formal code review process exists before committing any schema changes.
Emergency Changes: These can occur in urgent situations but must be documented, audited, and reconciled later.
Reconciliation Migrations: Post-emergency changes, a migration or script should revert the database to a known good state.

#### First Principles Solution
Primary Codebase:
Maintain all schema changes in a primary Git repository where they are reviewed, committed, and tagged with version numbers.
Emergency Schema Changes:
Use a separate branch for emergency changes (e.g., emergency-branch).
Post-Emergency Reconciliation:
After an emergency change is applied, create a script or migration to revert the schema back to the last reviewed state.
Audit and Documentation:
Document all emergency changes in a central log for future reference.
Detailed Steps

Primary Codebase Management:

All non-emergency schema changes should be committed to master (or any other primary branch) after successful reviews.
BASH
# Example commit message format
git commit -m "schema: add user table with columns [ID, name, email]"

Emergency Branch for Urgent Changes:

For urgent changes, a DBA can create an emergency branch and apply the necessary schema modifications.
BASH
# Clone repository and switch to emergency-branch
git clone https://github.com/your-repo/schema.git
cd schema
git checkout -b emergency-branch

Applying Emergency Changes:

The DBA applies the changes directly in this branch without further review.
SQL
# Example SQL change
ALTER TABLE users ADD COLUMN phone_number VARCHAR(15);

Post-Emergency Reconciliation:

After the emergency period, a migration script is run to revert the schema to the last known good state from master.
BASH
# Run reconciliation script
./scripts/reconcile-schema.sh

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that every schema change is traceable to a reviewed commit while allowing for emergency changes. The key challenge is ensuring that post-emergency reconciliations are thorough and accurate, maintaining the integrity of both the primary codebase and the audit trail.

#### Common mistakes
include:

Forgetting to document or log emergency changes.
Not thoroughly testing reconciliation scripts before deployment.
Skipping review processes even in non-emergency situations.
Common Follow-Ups
How would you ensure that DBAs don't bypass the primary review process?
What tools or integrations could be used to automate parts of this process, like schema versioning or migration testing?
How would you handle complex emergency changes that involve multiple tables and dependencies?

