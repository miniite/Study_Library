---
type: "practice"
difficulty: "Intermediate"
---

# How would you enforce that every service exposes a standard /health and /version endpoint via a CI check, so the deploy tooling can universally verify rollouts?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Validating Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To ensure every service exposes a /health and /version endpoint as part of its deployment process, we'll leverage CI/CD to enforce this through static checks using tools like curl. This approach ensures that any new or updated service adheres to these standards before deployment. Key steps include defining the check logic in a script, integrating it into the CI pipeline, and regularly updating the health and version endpoints.

#### Triaging the Question

The core of this question revolves around ensuring consistency across services by enforcing standard endpoint exposure through CI/CD checks. Specifically, the interviewer is probing your understanding of how to integrate static verification steps into the deployment process using tools like curl or similar scripts. You'll need to assume a basic setup where each service has its own CI/CD pipeline and that you have access to the build and release processes.

#### First-Principles Solution
Define the Check Logic: Create a script (e.g., in Bash) that uses curl to send requests to /health and /version endpoints.
Integrate into CI Pipeline: Modify your existing CI configuration files (like Jenkinsfile or GitLab CI/CD YAML) to include steps for running this script as part of the build process.
Set Up Regular Verification: Ensure that these checks are run not just during development but also on every code push, to catch any issues early.
Step-by-Step Solution:

Script Creation:

BASH
# health-check.sh
#!/bin/bash

set -e  # Exit immediately if a command exits with a non-zero status.

API_URL="http://localhost:8080"  # Change to the actual service URL in CI.

echo "Checking /health endpoint..."
curl -s -o /dev/null -w "%{http_code}" $API_URL/health | grep -q 200
if [ $? -ne 0 ]; then
    echo "Health check failed." >&2
    exit 1
fi

echo "Checking /version endpoint..."
curl -s $API_URL/version | jq . > version.json  # Using jq for JSON parsing.
if [ ! -f version.json ]; then
    echo "Version check failed." >&2
    exit 1
fi

Integrate into CI/CD Pipeline:

YAML
stages:
  - build
  - test

build:
  stage: build
  script:
    - ./build.sh

test:
  stage: test
  script:
    - ./health-check.sh
  only:
    refs:
      - master  # Adjust as needed.

Regular Verification: Ensure that these checks are part of both the development and production pipelines to catch any issues early.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

By integrating static verification steps using curl into your CI/CD pipeline, you can ensure that every service adheres to standard practices. Key points include defining clear check logic, integrating these checks into existing workflows, and ensuring continuous verification. Common mistakes might include neglecting to update the script with new endpoints as services evolve or not setting up the checks in all relevant environments.

#### Follow-up
questions could involve discussing alternative tools (like Istio for service mesh), more complex health checks involving multiple microservices, or integration testing scenarios.

**Code:**

```

#!/bin/bash

set -e # Exit immediately if a command exits with a non-zero status. API_URL="http://localhost:8080" # Change to the actual service URL in CI.

echo "Checking /health endpoint..." curl -s -o /dev/null -w "%{http_code}" $API_URL/health | grep -q 200 if [ $? -ne 0 ]; then echo "Health check failed." >&2 exit 1 fi

echo "Checking /version endpoint..." curl -s $API_URL/version | jq . > version.json # Using jq for JSON parsing. if [ ! -f version.json ]; then echo "Version check failed." >&2 exit 1 fi

CODE
Copy

```yaml
stages:
  - build
  - test

build:
  stage: build
  script:
    - ./build.sh

test:
  stage: test
  script:
    - ./health-check.sh
  only:
    refs:
      - master  # Adjust as needed.
```

