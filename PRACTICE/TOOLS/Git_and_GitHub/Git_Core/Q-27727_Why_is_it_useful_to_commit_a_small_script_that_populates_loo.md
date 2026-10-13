---
type: "practice"
difficulty: "Basic"
---

# Why is it useful to commit a small script that populates lookup tables (like country codes) as versioned reference data?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Versioning SQL and Migrations](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_SQL_and_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Committing a small script that populates lookup tables, like country codes, as versioned reference data is useful because it ensures consistency and traceability in your database schema. This practice helps maintain a clean codebase by separating configuration from application logic, and it facilitates easy updates and migrations.

#### Triaging the Question

The question is probing why committing such a script to Git (version control) and GitHub (cloud-based hosting service for Git repositories) is beneficial. The key points to consider are:

Traceability: Keeping track of changes in your database schema.

#### Cons
istency: Ensuring that all environments have the same data.
Collaboration: Facilitating team work by sharing and reviewing scripts.
Reusability: Making it easy to reuse the script across different projects or future development.
Step-by-Step Solution

Traceability

When you commit a script that populates lookup tables, every change in this data is recorded in your version control history. This means you can always go back and see what changes were made and when.
PLAINTEXT
# Example of committing a script to Git
git add populate_country_codes.sh
git commit -m "Add initial country code population script"

#### Cons
istency

By versioning your reference data, you ensure that all environments (development, staging, production) have the same set of lookup tables. This reduces the chances of discrepancies or bugs caused by different sets of data.
PLAINTEXT
# Example scenario where versioned data is critical
# In a development environment, you might accidentally use outdated country codes,
# leading to errors in your application logic.

Collaboration

When multiple developers are working on the same project, having a script for populating lookup tables in Git/GitHub allows them to share and review each other's work more effectively. This can lead to better code quality and faster development cycles.
PLAINTEXT
# Example of collaboration through GitHub pull requests
# Developer A proposes changes to country codes via a pull request,
# which Developer B reviews before merging into the main branch.

Reusability

Versioning lookup tables as scripts makes it easy to reuse them across different projects or future development cycles. You can easily copy and modify these scripts for new projects without having to manually recreate all the data.
PLAINTEXT
# Example of reusing a script in another project
# You find an existing script that populates country codes, and you decide to use it for a new application,
# adapting the code as necessary.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Committing scripts that populate lookup tables ensures consistency, traceability, collaboration, and reusability in your database development workflow. It's important to avoid the common mistake of hardcoding these values directly into your application logic or manually managing them across environments.

#### Common Mistakes

Hardcoding Data: Directly embedding data into SQL scripts can make it difficult to manage and update.

PLAINTEXT
# Bad practice: Hardcoded country codes in SQL script
INSERT INTO countries (code, name) VALUES ('US', 'United States');

Manual Data Management: Manually managing lookup tables across environments can lead to inconsistencies and errors.

PLAINTEXT
# Manual management: Different environments with different sets of data
Development: ('US', 'United Kingdom')
Staging:    ('US', 'Germany')
Production: ('US', 'France')
Triangulating the Question Further

Why not just use hardcoded values?

Hardcoded values can lead to inconsistencies and make it harder to maintain a clean codebase. By using scripts, you centralize the management of your data.

What if the script is too large or complex?

Even large or complex scripts can be versioned effectively. The benefits of traceability and consistency outweigh the potential complexity.

**Code:**

```bash
# Example of a simple country code population script
#!/bin/bash

echo "Populating country codes..."
echo "US,United States" >> countries.csv
echo "GB,United Kingdom" >> countries.csv
```

**Example input / output:**

```text
# Initial commit to populate country codes
Initial commit: Add initial country code population script

# After making changes and committing again
Commit 2: Update country code for United Kingdom
```

**Data:**

```plaintext
countries.csv
US,United States
GB,United Kingdom
```

