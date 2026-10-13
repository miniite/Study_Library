---
type: "practice"
difficulty: "Intermediate"
---

# You want git bisect to find the commit where an integration test started failing due to a schema change, but some intermediate commits have migrations that don't apply cleanly. How do you keep bisect meaningful with skip or a migrate-aware test script?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Git Bisect](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Bisect.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the commit where an integration test started failing due to a schema change, while dealing with intermediate commits that have migrations which don't apply cleanly, you can use git bisect in combination with some advanced techniques like skipping certain revisions or writing a migrate-aware test script. The goal is to keep the bisect process meaningful by avoiding false negatives and positives.

Firstly, let's understand what git bisect does: it quickly narrows down the range of commits between two known points (one good and one bad) by repeatedly checking the middle point in the commit history. However, when migrations fail, this can lead to false results since the test might fail not because of a schema change but due to migration errors.

To handle this effectively:

Skip Migrations: You can use git bisect skip for commits where the migrations don't apply cleanly. This tells Git that the current commit is good and skips it from further consideration.
Migrate-Aware Test Script: Write a test script that not only checks the schema but also ensures the migration processes successfully.

Here’s how you can approach this:

TL;DR

Use git bisect skip for problematic commits, and write a migrate-aware test to ensure both the schema and migrations are correctly applied before declaring a commit as bad.

#### Triaging the Question

The question is essentially about ensuring that Git Bisect works effectively by handling cases where migrations fail. The key points are:

Skip Commits: Some intermediate commits have migrations that don't apply cleanly.
False Positives/Negatives: If not handled correctly, these skipped commits can lead to incorrect results from git bisect.
Step-by-Step Solution

Setup Your Git Repository:

BASH
git checkout -b feature/bugfix
# Make sure you have the initial schema and test setup.

Identify the Range of Commits to Bisect: Suppose the integration tests started failing between commit a1 (good) and z1 (bad).

Perform Initial Bisect:

BASH
git bisect start a1 z1

Handle Migrations with git bisect skip: For commits where migrations fail, use git bisect skip. For example, if commit b2 has failed migrations:

BASH
# Check out the problematic commit
git checkout b2
# Manually apply the migration or mark it as skipped in the test script.
git bisect skip

Write a Migrate-Aware Test Script: Your test script should first ensure that all migrations have applied successfully before running the integration tests.

Here’s an example of what your test script might look like:

PYTHON
import os
from subprocess import check_call

# Function to run migrations
def apply_migrations():
    try:
        os.chdir("/path/to/migrations")
        check_call(["./migrate.sh"])  # Adjust the command as needed
        print("Migrations applied successfully.")
    except Exception as e:
        print(f"Migrations failed: {e}")
        return False

# Function to run integration tests
def run_integration_tests():
    try:
        os.chdir("/path/to/tests")
        check_call(["pytest"])  # Adjust the command as needed
        print("Integration tests passed.")
        return True
    except Exception as e:
        print(f"Tests failed: {e}")
        return False

if apply_migrations():
    run_integration_tests()
else:
    exit(1)  # Skip this commit in bisect

Continue Bisecting: After handling the problematic commits, continue running git bisect:

BASH
git bisect bad
git bisect good

#### Closing Summary
and Common Mistakes/Follow-Ups

By combining git bisect skip for troublesome migrations and a migrate-aware test script, you ensure that your bisect process remains meaningful. Common mistakes include:

Skipping too many commits without proper validation.
Not properly handling the intermediate states of migration success or failure.

#### Follow-ups
might involve optimizing the migration script further to handle complex schema changes or understanding how git bisect can be integrated with continuous integration pipelines for automated testing and skipping.

**Comparison:**

```
Step	Command	Description
1	git bisect start	Initialize git bisect between two known points (good and bad).
2	git bisect skip b2	Skip commit b2 if migrations fail.
3	./test_script.py	Run migrate-aware test script to ensure both schema and migrations are applied successfully.
4	git bisect bad	Mark the current commit as bad if tests failed.
5	git bisect good	Mark the next commit as good if no issues were found in the migration or tests.
```

**Code:**

```python
# Example test script
import os
from subprocess import check_call

def apply_migrations():
    try:
        os.chdir("/path/to/migrations")
        check_call(["./migrate.sh"])
        print("Migrations applied successfully.")
    except Exception as e:
        print(f"Migrations failed: {e}")
        return False

def run_integration_tests():
    try:
        os.chdir("/path/to/tests")
        check_call(["pytest"])
        print("Integration tests passed.")
        return True
    except Exception as e:
        print(f"Tests failed: {e}")
        return False

if apply_migrations():
    run_integration_tests()
else:
    exit(1)
```

**Example input / output:**

```text
git bisect start a1 z1

# Check out commit b2 and handle migrations
git checkout b2
./test_script.py

# Output:
# Migrations applied successfully.
# Traceback (most recent call last):
#   ...
# Exception: Tests failed due to migration errors.

# Use git bisect skip for problematic commits
git bisect skip

# Continue bisecting
git bisect bad  # Mark this as a bad commit
git bisect good  # Mark the next commit as good
```

**Data:**

```json
{
  "commits": [
    {"id": "a1", "status": "good"},
    {"id": "b2", "status": "migration failure"},
    {"id": "z1", "status": "bad"}
  ],
  "test_script_output": {
    "commit_id": "b2",
    "migrations_applied": false,
    "integration_tests_passed": null
  }
}
```

**Flow:**

1. No
2. Yes
3. Start Bisect
4. Initialize Bisect
5. Run Migrate-Aware Test
6. Test Passed?
7. Mark as Bad, Skip
8. Continue Bisecting
9. Mark Good and Continue

