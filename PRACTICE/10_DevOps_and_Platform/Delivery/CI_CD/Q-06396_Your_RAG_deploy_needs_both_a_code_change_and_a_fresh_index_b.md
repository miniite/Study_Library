---
type: "practice"
difficulty: "Deep"
---

# Your RAG deploy needs both a code change and a fresh index build; how do you order them so users never query new code against a stale index or old code against a new index?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Expand and Contract Migrations](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deploying an RAG (Retrieval-Augmented Generation) system, we must ensure that both our codebase and data indexing are in sync to avoid serving users with mismatched results. This problem requires us to carefully coordinate two processes: applying the latest code changes and building a fresh index from scratch. The key is to sequence these operations such that they do not overlap or conflict.

Firstly, let's reframe the question by stating what it’s probing:

Understanding of Dependency Management: Ensuring that dependencies between components are correctly managed.
Concurrency Handling: Preventing race conditions where code changes and index updates might intersect.
System Resilience: Maintaining system reliability during deployments.

To start, let's make a few assumptions:

We have a CI/CD pipeline in place for our RAG application.
The deployment process involves updating both the codebase and rebuilding the data index.
Both operations need to be atomic or isolated from each other as much as possible.
Step-by-Step Solution

Code Change Deployment:

We'll use a staging environment where new code is deployed first, isolated from production traffic.
After deployment, we can run automated tests (unit, integration, end-to-end) to ensure the new code works correctly in isolation with the existing index.

Index Build and Testing:

Once the staging environment passes all tests, we start the process of building a fresh data index based on the latest code.
This step might be resource-intensive, so we need to manage it carefully, possibly by using scheduled jobs or dedicated infrastructure.
After indexing is complete, we verify that this new index works with the current production codebase. This can involve running query simulations against both the old and new indexes to ensure consistency.

Switching to New Code and Index:

If all tests pass in the staging environment, we proceed to deploy the updated code to the production environment.
Simultaneously, we switch over to the newly built index. This can be achieved through a rolling update strategy where nodes are updated one at a time.

Final Checks and Monitoring:

Once both the new code and the fresh index are live in production, perform final manual checks to ensure everything is functioning as expected.
Set up monitoring to alert us if any issues arise post-deployment.
Full Solution

To implement this, we can use a combination of tools and strategies:

CI/CD Pipeline:

Configure your CI tool (e.g., Jenkins, GitLab CI) to build the new code and run tests.
Integrate an indexing service that rebuilds indexes based on the latest code commit.

Deployment Strategy:

Use a blue-green deployment approach where we have two identical environments (blue for current production, green for updates).
Deploy new code to the green environment first, test thoroughly, then switch traffic from blue to green.
Simultaneously, build and swap out the index on the green environment.

Indexing Service:

Ensure your indexing service supports incremental builds if possible, reducing the load during deployment.
Use version control for your data schema to manage changes smoothly.

#### Closing Summary

The key takeaway is that by carefully managing the sequence of code change and index build operations, we can ensure a seamless transition without serving stale or inconsistent results. This approach requires robust testing and monitoring but ensures high availability and reliability of our RAG system during deployments.

#### Common Mistakes

Forgetting to thoroughly test in staging before switching production.
Not properly isolating the index update process from live queries.
Failing to manage resources efficiently, leading to downtime or performance issues.

**Comparison:**

```
Time	Event
15:30	Deploy new code to staging
16:00	Build fresh index in staging
17:00	Switch production to new code
```

**Code:**

```python
# Example of a simplified CI/CD pipeline script snippet for RAG deployment

import os
from datetime import datetime

def deploy_code(branch):
    # Stage new code changes in a staging environment
    print(f"Deploying {branch} branch to staging at {datetime.now()}")

def test_staging():
    # Run tests on the staged environment
    print("Running unit, integration, and end-to-end tests...")

def build_index(branch):
    # Build fresh index based on the latest code changes
    print(f"Building new index for {branch} branch at {datetime.now()}")

def switch_production(new_code_branch, new_index_version):
    # Switch production to use the new code and index version
    print(f"Switching production to use {new_code_branch} code and {new_index_version} index at {datetime.now()}")
```

**Example input / output:**

```text
Staging Environment:
- Deployed branch: develop
- Run tests on staging

Production Environment (Initial State):
- Code version: v1.0.0
- Index version: v42

Deployment Steps:

1. Staging -> Deploy new code changes to develop branch.
   - New code deployed at 15:30 UTC
2. Test Staging:
   - All tests passed successfully.
3. Build Fresh Index:
   - New index built for develop branch at 16:00 UTC
4. Switch Production:
   - Prod switched to use new develop code and v43 index at 17:00 UTC

Final State:
- Code version: v2.0.0 (develop)
- Index version: v43
```

**Data:**

```yaml
# Sample data schema for indexing service

index_schema:
  - name: doc_id
    type: string
  - name: content
    type: string
  - name: metadata
    type: object
      title: Metadata
      properties:
        date: datetime
        author: string
```

**Flow:**

1. Staging Code Deployment
2. Tests
3. Build New Index
4. Switch Production

