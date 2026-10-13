---
type: "practice"
difficulty: "Intermediate"
---

# How would you bake a git commit hash and pipeline-run ID into the metadata/audit columns of the tables your pipeline writes, so you can trace which deploy produced which rows?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pipeline Triggers and Path Filters](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_Triggers_and_Path_Filters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that each row in our database tables is tied to its specific CI/CD deployment and pipeline run, we need to bake the git commit hash and pipeline-run ID into the metadata columns of the data being written. This allows us to trace back exactly which code version and pipeline execution produced any given set of rows.

#### Triage
Understanding the Question

The core of this question revolves around embedding unique identifiers (git commit hashes and pipeline-run IDs) into database tables during a CI/CD pipeline run. The main objectives are:

Uniqueness: Each row should be uniquely linked to a specific code version and deployment.
Traceability: Being able to identify the exact git commit and pipeline run that generated each batch of data.
Automation: Ensuring this process is automated within our CI/CD workflow.

#### First Principles Solution

To achieve this, we will:

Capture Metadata: Extract the current git commit hash and pipeline-run ID at runtime.
Embed Metadata: Append these metadata values to every row being inserted into the database.
Store Metadata: Ensure that the metadata is stored in a way that it can be queried later for traceability.
Step-by-Step Implementation
1. Capturing Git Commit Hash

We will use Git commands or environment variables to capture the current commit hash:

BASH
git rev-parse HEAD

This command returns the current git commit hash.

2. Determining Pipeline Run ID

The pipeline-run ID can be obtained from a CI/CD tool like Jenkins, GitLab CI, or GitHub Actions. For instance, in Jenkins, we can use:

GROOVY
env.BUILD_NUMBER

This gives us the unique build number for each pipeline run.

3. Embedding Metadata into Rows

When writing data to the database, we will append the commit hash and pipeline-run ID as metadata columns. Assuming our table structure includes commit_hash and pipeline_run_id, the SQL might look like this:

SQL
INSERT INTO your_table (column1, column2, ..., commit_hash, pipeline_run_id)
VALUES ('value1', 'value2', ..., (SELECT git.rev_parse('HEAD')), env.BUILD_NUMBER);

This ensures that each row is tagged with its respective metadata.

4. Automating the Process

To automate this embedding process, we can write a script that runs as part of our CI/CD pipeline:

BASH
#!/bin/bash

# Get git commit hash
COMMIT_HASH=$(git rev-parse HEAD)

# Get pipeline run ID (example for Jenkins)
PIPELINE_RUN_ID=$BUILD_NUMBER

# Write data to database with metadata
mysql -e "INSERT INTO your_table (column1, column2, ..., commit_hash, pipeline_run_id) VALUES ('value1', 'value2', ..., $COMMIT_HASH, $PIPELINE_RUN_ID);"

#### Closing Summary
and Common Mistakes/Follow-Ups

By embedding the git commit hash and pipeline-run ID into our database rows, we ensure traceability and accountability in our data processing. Key points to consider include:

#### Cons
istency: Ensuring that metadata is always captured and embedded.
Performance: The impact of additional columns on insert performance.
Security: Safeguarding sensitive information like commit hashes if they are part of public repositories.

#### Common mistakes
include neglecting the capture or embedding step, leading to loss of traceability. Another mistake is overcomplicating the metadata handling process without considering performance implications.

#### Follow-Ups
How would you handle data drift and ensure that historical data retains its metadata?
What if multiple pipeline runs are writing data simultaneously?

**Comparison:**

| id | column1 | column2 | commit_hash | pipeline_run_id |
|----|---------|---------|-------------|----------------|
| 1  | value1  | 123     | abcdef01... | 456            |

**Code:**

```bash
#!/bin/bash

COMMIT_HASH=$(git rev-parse HEAD)
PIPELINE_RUN_ID=$BUILD_NUMBER

mysql -e "INSERT INTO your_table (column1, column2, ..., commit_hash, pipeline_run_id) VALUES ('value1', 'value2', ..., $COMMIT_HASH, $PIPELINE_RUN_ID);"
```

**Example input / output:**

```text
Sample input:
- `column1 = 'value1'`
- `column2 = 123`

Output:
- Row inserted with values `('value1', 123, 'commit_hash_value', 456)`
```

**Data:**

```sql
CREATE TABLE your_table (
    id INT AUTO_INCREMENT,
    column1 VARCHAR(255),
    column2 INT,
    commit_hash CHAR(40),
    pipeline_run_id INT,
    PRIMARY KEY (id)
);
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Git Commit Hash Available?
5. Extract Git Commit Hash
6. Use Default Value
7. Embed Metadata in Row
8. Embed Metadata in Row
9. Insert Data into Database
10. End

