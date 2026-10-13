---
type: "practice"
difficulty: "Basic"
---

# Why might a data team review the SQL and config of a pipeline change in a pull request even though the actual job runs on a managed Airflow or warehouse cluster?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a data team reviews changes to their pipelines, whether it's an SQL query or configuration settings, they're ensuring that the logic and structure are correct before deployment. This process is crucial even if these changes will be run on managed Airflow or warehouse clusters for several reasons.

Firstly, the review helps catch any potential errors early in the development lifecycle. A small mistake can lead to significant issues down the line, especially in complex data pipelines. For instance, a typo in an SQL query could result in incorrect data being loaded into the warehouse, leading to faulty analytics or reports.

Secondly, the pull request (PR) review process allows team members with different perspectives to provide feedback and suggestions for improvements. This ensures that the code adheres to best practices and standards maintained by the team.

Thirdly, reviewing changes helps maintain consistency across the pipeline. It ensures that all parts of the system are up-to-date and follow a consistent approach, which is particularly important in large-scale data pipelines where multiple components interact with each other.

Finally, it’s good practice to have an audit trail for any changes made. This makes it easier to trace back issues if something goes wrong post-deployment and helps with version control and change management.

**Code:**

```python
# Example SQL query in a PR
query = """
INSERT INTO users_table (user_id, name, email)
SELECT id, first_name || ' ' || last_name AS full_name, lower(email)
FROM imported_users;
"""
```

