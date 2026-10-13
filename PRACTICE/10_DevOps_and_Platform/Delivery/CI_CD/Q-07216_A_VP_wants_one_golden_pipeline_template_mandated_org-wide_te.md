---
type: "practice"
difficulty: "Expert"
---

# A VP wants one golden pipeline template mandated org-wide; teams route around mandates. How do you make the paved road so good that adoption is voluntary, and how do you fund and measure that?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Pipelines and Templates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Golden_Pipelines_and_Templates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of ensuring a single CI/CD pipeline template (the "golden pipeline") is adopted voluntarily across an organization, we need to focus on creating a highly reliable, efficient, and user-friendly system that teams naturally gravitate towards. This involves several key steps—designing the pipeline, funding its development, and measuring its success.

Firstly, let's define what makes a golden CI/CD pipeline template compelling. A good pipeline should be:

Extremely Reliable: Minimal downtime and high availability.
Efficient: Fast and resource-friendly to run.
User-Friendly: Easy to configure and extend without extensive coding knowledge.
Integrated with DevOps Best Practices: Continuous Integration, Automated Testing, and Deployment.

#### Triage
What the Question is Probing

The VP's mandate indicates a need for standardization but also acknowledges resistance from teams who may find it inconvenient or unnecessary. The core issue is to make this template so appealing that teams choose it voluntarily over their current processes. This involves technical, organizational, and financial considerations.

#### First-Principles Solution
1. Design the Pipeline

The golden pipeline should be a streamlined version of the best CI/CD practices. Start by identifying common workflows across different projects:

Code Repository
Build
Test
Deploy
Monitor

Each stage should have clear, automated steps that include linting, unit tests, integration tests, and deployment. Use modern tools like Jenkins X, GitLab CI, or GitHub Actions to implement this.

2. Ensure Reliability

Implement robust monitoring and alerting systems using tools like Prometheus and Grafana. Regularly audit the pipeline for any issues and fix them promptly. Set up a zero-downtime deployment strategy with blue-green deployments or canary releases.

3. Efficiency

Optimize the pipeline to minimize resource usage and maximize speed. This includes:

Using just-in-time (JIT) builds.
Implementing caching strategies for dependencies.
Leveraging managed services for CI/CD tools.
4. User-Friendly Design

Make sure the pipeline is easy to configure by providing clear documentation, templates, and pre-built configurations. Offer a graphical interface or command-line tool that abstracts away complex settings.

5. Training and Support

Provide comprehensive training sessions and support channels for teams. This can be done through internal workshops, webinars, and documentation centers.

Funding the Pipeline

To fund the development of this pipeline:

Internal Budget: Allocate a portion of the existing DevOps budget towards improving the pipeline.
Resource Pooling: Create a shared resource pool where each team contributes to maintaining the pipeline infrastructure.
Vendor Sponsorship: Partner with CI/CD tool vendors who can provide free or discounted licenses in exchange for promoting their tools.
Measuring Success

To measure the success of the golden pipeline:

Deployment Frequency and Speed: Track how often teams deploy code and how quickly they can do so.
Test Coverage: Monitor test coverage metrics to ensure comprehensive testing is being done.
Code Quality: Use static analysis tools like SonarQube to track code quality improvements.
Team Satisfaction Surveys: Regularly survey teams on their satisfaction with the pipeline and suggest areas for improvement.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By focusing on reliability, efficiency, user-friendliness, comprehensive documentation, and robust support, we can create a CI/CD pipeline that naturally appeals to teams. However, common pitfalls include underestimating the need for ongoing maintenance and neglecting user feedback. Regularly engaging with teams to gather insights will help ensure the pipeline remains relevant and effective.

#### Common mistakes

Ignoring user needs in favor of theoretical perfection.
Failing to provide adequate training and support.
Not setting up comprehensive monitoring and alerting systems.

#### Follow-ups
might involve diving deeper into specific tools, exploring alternative deployment strategies, or discussing how to handle legacy projects that may not fit the new pipeline structure.

**Comparison:**

```
Metric	Current State	Target
Deployment Frequency	Once per week	Multiple times per day
Code Coverage	30%	100%
Test Execution Time	Long and manual	Short and automated
```

**Code:**

```yaml
# Example of a simple GitLab CI configuration for a Node.js project
image: node:14

stages:
  - build
  - test
  - deploy

variables:
  NODE_ENV: production

before_script:
  - npm install -g npm@6
  - npm ci

build:
  stage: build
  script:
    - npm run build

test:
  stage: test
  script:
    - npm run test

deploy:
  stage: deploy
  script:
    - echo "Deploying to production"
```

**Example input / output:**

```text
Input:
- Team A has a Node.js project that currently uses manual deployments and lacks automated tests.

Output:
- Team A adopts the new CI/CD pipeline for their project.
- Deployment frequency increases from once per week to multiple times per day.
- Test coverage improves, with 100% code coverage achieved after a few iterations.
```

**Data:**

```json
{
  "pipeline": {
    "stages": ["build", "test", "deploy"],
    "variables": {"NODE_ENV": "production"},
    "before_script": [
      "npm install -g npm@6",
      "npm ci"
    ],
    "build": {
      "stage": "build",
      "script": ["npm run build"]
    },
    "test": {
      "stage": "test",
      "script": ["npm run test"]
    },
    "deploy": {
      "stage": "deploy",
      "script": ["echo 'Deploying to production'"]
    }
  }
}
```

**Flow:**

1. Code Commit
2. Test
3. Build
4. Deploy
5. Monitor

