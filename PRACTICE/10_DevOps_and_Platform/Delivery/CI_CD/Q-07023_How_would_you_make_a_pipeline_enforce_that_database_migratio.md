---
type: "practice"
difficulty: "Intermediate"
---

# How would you make a pipeline enforce that database migrations are reviewed by a DBA/senior for tables above a size threshold, while letting small tables auto-proceed, purely through automation?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Database Migrations in Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Database_Migrations_in_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that database migrations for large tables are reviewed by a DBA before deployment, we can leverage CI/CD pipelines to automate this process. The key idea is to integrate size checks within our CI pipeline and route migrations based on the table sizes. Here’s how I would approach it:

Understand What the Question Asks: We need to design a CI/CD step that evaluates the size of database tables involved in migrations, and routes them either for automatic deployment or manual review by a DBA.

#### Assumptions

The migration process is part of a continuous integration pipeline.
We have a mechanism to track table sizes (e.g., through schema definitions).
There’s an existing CI/CD tool like Jenkins, GitLab CI, or GitHub Actions.

#### First-Principles Solution

Define the threshold size for tables that require manual review.
During the migration process, check if any table exceeds this size.
If a large table is detected, halt the pipeline and send an alert to a specific group (DBAs) or a dedicated Slack channel.
Smaller tables should automatically proceed without additional checks.

#### Implementation
Steps:

Table Size Calculation:

Use a pre-migration step to calculate the size of all tables involved in the migration. This can be done by querying the database schema and computing the row count and estimated storage requirements.

Pipeline Configuration:

Integrate this calculation into our CI/CD pipeline as part of the pre-deployment checks.
Use scripting languages like Python or Bash to automate these calculations.

Decision Logic:

Implement logic in the pipeline to compare table sizes against a predefined threshold.
If any large tables are detected, halt the pipeline and trigger manual review.

Notification Mechanism:

Set up notifications (e.g., emails or Slack messages) when the size check fails.

#### Example
Implementation

Here’s an example using GitLab CI:

YAML
stages:
  - build
  - test
  - deploy

before_script:
  - pip install pandas psycopg2-binary

build_migration_check:
  stage: build
  script:
    - python3 ./check_table_sizes.py --threshold=1000000 # Example threshold
    - if [ $? -ne 0 ]; then exit 1; fi

deploy_small_tables:
  stage: deploy
  when: on_success
  only:
    - main
  script:
    - python3 ./run_migrations.py --tables=small_tables.csv
  dependencies:
    - build_migration_check

deploy_large_tables_review:
  stage: deploy
  when: manual
  rules:
    - if: '$CI_COMMIT_REF_NAME == "main"'
      changes:
        - schemas/*
      when: always
  script:
    - python3 ./run_migrations.py --tables=large_tables.csv

#### Common Mistakes
and Follow-Ups:
Mistake: Overlooking the need for accurate table size calculations.
Mistake: Not properly handling edge cases where tables are large but critical (e.g., historical data).

#### Closing Summary

By integrating automated checks into our CI/CD pipeline, we ensure that only small tables automatically proceed while large ones require manual review by a DBA. This approach balances automation with quality assurance and reduces the risk of deploying poorly reviewed migrations.

**Comparison:**

```
Table Name	Rows
small_table1	500
large_table2	1500000
Notification	Content
Email	Large table detected: large_table2. Manual review required.
Slack
- large_table2 needs manual review.
```

**Code:**

```yaml
stages:
  - build
  - test
  - deploy

before_script:
  - pip install pandas psycopg2-binary

build_migration_check:
  stage: build
  script:
    - python3 ./check_table_sizes.py --threshold=1000000 # Example threshold
    - if [ $? -ne 0 ]; then exit 1; fi

deploy_small_tables:
  stage: deploy
  when: on_success
  only:
    - main
  script:
    - python3 ./run_migrations.py --tables=small_tables.csv
  dependencies:
    - build_migration_check

deploy_large_tables_review:
  stage: deploy
  when: manual
  rules:
    - if: '$CI_COMMIT_REF_NAME == "main"'
      changes:
        - schemas/*
      when: always
  script:
    - python3 ./run_migrations.py --tables=large_tables.csv
```

**Example input / output:**

```text
Input: Threshold = 1000000, Tables = ['small_table1', 'large_table2']
Output:
- Large Table Detected: large_table2
- Pipeline Halted and Notification Sent
```

**Data:**

```python
import pandas as pd

def check_table_sizes(threshold):
    # Query database for table sizes
    df = pd.read_sql_query("SELECT * FROM information_schema.tables", conn)
    table_sizes = df.groupby('table_name').agg({'rows': 'count'}).reset_index()

    large_tables = table_sizes[table_sizes['rows'] > threshold]
    if not large_tables.empty:
        return False, large_tables
    return True, None

def run_migrations(tables):
    # Run migrations on specified tables
    for table in tables:
        print(f"Running migration for {table}")
```

**Flow:**

1. Small Tables
2. Large Tables
3. Start Pipeline
4. Check Table Sizes
5. Deploy Automatically
6. Manual Review Required
7. Send Notification
8. Halt Pipeline

