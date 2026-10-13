---
type: "practice"
difficulty: "Basic"
---

# When someone says 'it works on my machine', how does CI help catch that problem?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ CI vs Continuous Delivery vs Deployment](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_vs_Continuous_Delivery_vs_Deployment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a developer says, "It works on my machine," it often means that their local environment differs from the deployment environment in some way, leading to discrepancies when the code is deployed. Continuous Integration (CI) helps catch these issues by ensuring that the codebase is built and tested automatically whenever changes are pushed to version control. This process standardizes the development and testing environments, making sure that what works locally also works on the server.

#### Triaging the Question

The core of this question is about understanding how CI can mitigate the "works on my machine" problem. The interviewer might be probing your knowledge of the different stages in a CI/CD pipeline and how they help maintain consistency across environments. You should assume that you are familiar with basic CI concepts, such as build, test, and deployment.

#### First-Principles Solution
Build Automation: In a CI setup, every time changes are committed to version control (like Git), an automated build process is triggered. This ensures that the application is built consistently across all environments.
Test Automation: Automated tests run as part of the CI pipeline to verify that the code works as expected before deployment. These include unit tests, integration tests, and end-to-end tests.
Environment Consistency: The development environment used by developers should closely mirror the production environment. This reduces the likelihood of discrepancies when the application is deployed.
Detailed Explanation

Build Automation: When a developer checks in code, an automated system (like Jenkins or GitLab CI) starts a build process. This ensures that everyone uses the same set of tools and dependencies, reducing the chances of issues arising from missing packages or configuration differences.

BASH
# Example: Running a build script triggered by a commit to GitHub
git clone https://github.com/username/repository.git
cd repository
./scripts/build.sh

Test Automation: Automated tests are run as part of the build process. This includes static code analysis, unit testing, and integration testing. These checks help catch bugs early in the development cycle.

BASH
# Example: Running tests after a successful build
./scripts/test.sh

Environment Consistency: The CI system can also be used to deploy the application to staging or pre-production environments that are configured similarly to production. This allows developers to see how the code behaves in a realistic setting before it goes live.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, CI helps by automating the build and test processes, ensuring consistency across development and production environments. Common mistakes include not having a well-defined CI pipeline or using outdated tools that don’t fully automate the process. Follow-up questions might explore specific tools used in CI (like Jenkins, GitLab CI) or how to handle complex multi-environment setups.

**Code:**

```bash
# Example build and test script
if [ -z "$CI" ]; then
  echo "Building locally"
  ./scripts/build.sh
else
  echo "Running in CI environment"
fi

echo "Running tests..."
./scripts/test.sh
```

**Flow:**

1. Code Commit
2. Trigger Build
3. Run Tests
4. Push to Staging
5. Deploy to Production (ifsuccessful)

