---
type: "practice"
difficulty: "Deep"
---

# How would you promote a machine-generated dbt/SQL artifact so the exact compiled SQL that ran in staging is byte-identical to what runs in prod, and what breaks that guarantee?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Build Reproducibility and Artifact Promotion](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/Build_Reproducibility_and_Artifact_Promotion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the exact compiled SQL from staging matches the production environment in a CI/CD pipeline, we need to follow several best practices and address potential pitfalls that might cause discrepancies between environments. The core idea is to maintain consistency across environments by ensuring the same codebase is used and executed identically.

#### Triaging the Question

The question asks how to promote an artifact (such as a dbt model or SQL script) so that the compiled SQL exactly matches in both staging and production environments. The key aspects being probed include:

Code Consistency: Ensuring the source code remains consistent across different environments.
Environment Isolation: Preventing accidental modifications to environment-specific configurations or metadata.
Build Process: Guaranteeing that the same build process is followed in both environments.
Testing and Validation: Implementing robust testing mechanisms to catch any discrepancies early.

#### First-Principles Solution

Code Base Management:

Use a version control system (e.g., Git) for all code artifacts.
Ensure that the staging environment uses the exact same branch or commit as production.

Environment Configuration:

Keep environment-specific configurations separate from the main codebase using configuration management tools like YAML files.
Define environment variables to handle differences, ensuring they are not accidentally modified in one environment but used correctly in another.

Build and Deployment Pipeline:

Implement a CI/CD pipeline that automates the build process, generating SQL artifacts consistently across environments.
Use pre-deployment checks (e.g., unit tests, integration tests) to validate that the generated SQL is identical between environments.

Logging and Monitoring:

Deploy logging and monitoring tools to track any differences in query execution plans or results.
Set up alerts for unexpected changes in query behavior or performance metrics.

#### Common Mistakes
and Follow-Ups
Manual Changes: Accidental manual changes in the environment-specific configurations can lead to discrepancies.
Configuration Drift: Configuration drift between environments, where settings are subtly different but not caught by tests.
Dependency Issues: Dependencies might vary slightly across environments, leading to minor differences in query results or performance.

#### Closing Summary

By maintaining a rigorous CI/CD pipeline that enforces consistent codebase usage and thorough testing, we can ensure that the exact compiled SQL from staging is byte-identical to what runs in production. Regular monitoring and logging help catch any issues early, making it easier to identify and correct configuration drift or dependency mismatches.

**Comparison:**

| Environment | Target Host | Threads |
|-------------|------------|---------|
| Staging     | staging_db_host   | 8       |
| Prod        | prod_db_host     | 16      |

**Code:**

```python
# Example of a simple CI/CD pipeline step for dbt models using GitHub Actions
name: Run DBT Models

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build_and_test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.8'

    - name: Install dependencies
      run: |
        pip install dbt-core
        pip install dbt-postgres  # or other relevant profiles

    - name: Run tests
      run: dbt test

    - name: Build and deploy
      run: dbt build --target staging && dbt deploy --target prod
```

**Example input / output:**

```text
# Example input-output for a dbt model
Input (Staging DB Schema):
- sales_orders (table)
- customers (table)

Output:
SELECT
    c.customer_id,
    SUM(so.quantity * so.price) as total_spent
FROM
    sales_orders so
JOIN
    customers c ON c.id = so.customer_id
GROUP BY
    c.customer_id
```

**Data:**

```yaml
# Example YAML configuration for environment-specific settings
staging:
  target: "postgresql://user@staging_db_host:5432/db"
  threads: 8

prod:
  target: "postgresql://user@prod_db_host:5432/db"
  threads: 16
```

**Flow:**

1. Code Repository
2. CI/CD Pipeline Trigger
3. GIT Checkout Code
4. Setup Python Environment
5. Install Dependencies
6. Test Models
7. Build and Deploy Models
8. Staging Build
9. Prod Deployment

