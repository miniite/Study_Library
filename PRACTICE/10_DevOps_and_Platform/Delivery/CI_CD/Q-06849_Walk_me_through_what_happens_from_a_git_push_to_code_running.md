---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through what happens from a git push to code running in production.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CI CD Pipeline Fundamentals](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_CD_Pipeline_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you push changes to your Git repository, the CI/CD pipeline takes over and orchestrates the process of building, testing, and deploying those changes. The journey from a git push to code running in production is a well-defined sequence that involves several automated steps.

#### Triaging the Question

The core of this question is to understand the end-to-end flow of software deployment using CI/CD practices. The interviewer might be probing your knowledge on how different tools and processes work together, as well as your understanding of potential issues and best practices. Common assumptions include a modern development environment with continuous integration (CI) and continuous delivery (CD).

From First Principles
Triggering the Pipeline: A git push to a designated branch or tag in a Git repository acts as a trigger for the CI/CD pipeline.
Build Stage: The build stage compiles source code into executable artifacts, such as binaries, Docker images, or containerized applications.
Test Stage: Automated tests are run to ensure that the new changes do not break existing functionality. This includes unit tests, integration tests, and end-to-end tests.
Deploy Stage: The successful build and tests result in deploying the application to a staging environment where it is tested further before going live.
Promote to Production: If the staging environment passes all tests, the code is promoted to production.
Detailed Walkthrough
1. Triggering the Pipeline

When you push changes to your Git repository, typically a webhook or other trigger in your CI/CD tool (e.g., Jenkins, GitLab, GitHub Actions) detects this change and starts the pipeline.

BASH
# Example of pushing code to a Git repository
git push origin main
2. Build Stage

The build stage runs commands necessary to compile and package the application. For example, if you're using Docker with a Dockerfile, the following might be part of your CI configuration:

YAML
- name: Build Docker image
  run: docker build -t myapp .
3. Test Stage

Automated tests are executed to ensure that new changes do not introduce bugs or break existing functionality.

YAML
- name: Run Unit Tests
  run: npm test

- name: Run Integration Tests
  run: npx mocha
4. Deploy Stage

If the tests pass, the application is deployed to a staging environment for further testing. This could be an automated process using tools like Kubernetes or AWS Elastic Beanstalk.

YAML
- name: Deploy to Staging Environment
  run: kubectl apply -f deployment-staging.yaml
5. Promote to Production

Once the staging environment is stable, you can promote the changes to production by updating a specific label or by manually triggering a promotion step in your pipeline.

YAML
- name: Promote to Production
  run: kubectl apply -f deployment-production.yaml --label version=1.0.1

#### Closing Summary
and Common Mistakes/Follow-Ups

The key takeaway is that the CI/CD process ensures code quality through automated testing before it reaches production, significantly reducing risks. Common mistakes include not having robust test coverage or failing to properly manage environment variables and secrets.

#### Follow-up
questions might include:

How do you handle security in your CI/CD pipeline?
What are some best practices for managing environment variables?
How do you ensure that your staging environment closely mimics production?

**Comparison:**

```
Stage	Action
Build	Compile and package the application
Test	Run unit, integration, and end-to-end tests
Deploy Staging	Deploy to a staging environment for further testing
Promote to Prod	Update production labels or configurations
```

**Flow:**

1. Code Push
2. CI/CD Trigger
3. Build Artifact
4. Test Automation
5. Staging Deployment
6. Promote to Production
7. Production Run

