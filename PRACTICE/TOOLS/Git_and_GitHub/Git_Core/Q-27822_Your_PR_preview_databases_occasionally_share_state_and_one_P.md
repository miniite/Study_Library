---
type: "practice"
difficulty: "Intermediate"
---

# Your PR preview databases occasionally share state and one PR's migration corrupts another's fixtures. From a Git/CI-config standpoint, how would you isolate per-PR database resources and clean them up deterministically?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Isolating CI Environments](<../../../../TOOLS/Git_and_GitHub/Git_Core/Isolating_CI_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with multiple database migrations across different Pull Requests (PRs), it’s crucial to ensure that each PR has its own isolated environment. This prevents state corruption or interference between PRs, which can lead to bugs and deployment issues. From a Git/CI-config standpoint, you need to set up your Continuous Integration (CI) pipeline in such a way that database resources are created and destroyed deterministically for each PR.

#### Triage

The question is asking how to isolate per-PR database resources so that they don't interfere with one another. This involves understanding the CI/CD lifecycle and ensuring that the databases used by different PRs are uniquely managed and cleaned up after use.

#### Assumptions
You have access to a Git repository for your application.
Your CI system supports environment variables or other methods of dynamic configuration.
The database migrations are run as part of your CI/CD pipeline.

#### First-Principles Solution
1. Environment Configuration

Each PR should be configured with unique identifiers, such as environment variables that specify the database connection string. These strings can include a unique suffix or prefix to distinguish between different environments.

SH
export DATABASE_URL="postgres://user:pass@localhost/dbname-pr-<PR_ID>"
2. CI Pipeline Configuration

In your CI configuration file (e.g., Jenkinsfile, .github/workflows/main.yml), you can parameterize the database connection string based on the PR ID or a similar unique identifier.

YAML
# .github/workflows/main.yml snippet
jobs:
  - name: Run Migrations and Tests
    runs-on: ubuntu-latest
    env:
      DATABASE_URL: "postgres://user:pass@localhost/dbname-pr-${{ github.event.pull_request.number }}"
    steps:
      - uses: actions/checkout@v2
      - run: echo "DATABASE_URL=$DATABASE_URL" >> .env
      - name: Install Dependencies
        run: npm install
      - name: Run Migrations and Tests
        run: npm run db:migrate && npm test
3. Database Initialization and Cleanup

For each PR, you need to ensure that a fresh database is created before the migrations are applied and then cleaned up afterward. This can be achieved by using Docker containers or cloud services like AWS RDS or Google Cloud SQL.

YAML
# .github/workflows/main.yml snippet for cleanup
  - name: Clean Up Database (Optional)
    if: github.event_name == 'pull_request' && github.action == 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: echo "Cleaning up database..."
4. Dynamic Environment Management

Use tools like Docker Compose to manage the lifecycle of your databases dynamically. You can create a docker-compose.yml file that spins up a new instance for each PR.

YAML
# docker-compose.yml snippet
version: '3'
services:
  db:
    image: postgres:latest
    environment:
      POSTGRES_DB: dbname-pr-${{ env.PULL_REQUEST_ID }}

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to isolate per-PR database resources, you need a combination of unique configuration identifiers, dynamic CI/CD pipeline setup, and proper cleanup mechanisms. Common mistakes include not properly configuring environment variables or forgetting to clean up resources after use.

For follow-ups:

What tools would you recommend for managing Docker containers in this context?
How would you handle database backups during the lifecycle of a PR?
Are there any trade-offs between using local database instances versus cloud services?

