---
type: "practice"
difficulty: "Intermediate"
---

# How does deploying the previous version differ from writing new code to undo a change?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Schema Changes in Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Schema_Changes_in_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In CI/CD, deploying the previous version and writing new code to undo a change are two distinct processes with different implications. When you deploy the previous version, you're essentially reverting your application state to how it was at a specific point in time. Writing new code to undo a change involves crafting custom code that undoes the effects of the existing implementation.

Let's break this down from first principles:

#### Triaging the Question

The question is probing understanding of CI/CD practices, specifically around version control and rollback mechanisms. We need to consider the implications on system state, maintainability, and the overall CI/CD pipeline.

#### Assumptions
Version Control System (VCS): We are using Git as our VCS.
CI/CD Pipeline: The pipeline includes automated tests, build processes, and deployment steps.

#### First-Principles Solution

When you deploy the previous version:

State Reversion:

You checkout to an existing commit that represents a previous version of your codebase.
This is typically done using Git commands like git checkout <commit-hash> or by triggering a pipeline step that checks out this specific version.

Maintenance and Risk:

This approach can be risky if the older version had bugs or issues not caught in testing.
It may lead to inconsistencies between different environments unless handled carefully.

When you write new code to undo a change:

Custom Code Development:

You need to understand the change that was introduced and develop custom code to revert it.
This could involve writing scripts, modifying existing functions, or adding new ones to achieve the desired outcome.

Testing and Validation:

The new code must be thoroughly tested in a staging environment before being deployed to production.
Manual testing and automated tests should verify that the change has been successfully undone without introducing new issues.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Deploying the previous version is straightforward but comes with risks. Writing custom code can provide more control and flexibility, but it requires thorough understanding and testing. Common mistakes include not thoroughly testing the rollback or failing to manage environment states correctly.

#### Follow-up
questions could explore how to implement automated rollbacks in a CI/CD pipeline, or discuss strategies for managing different versions of your application.

**Code:**

```bash
# Example Git commands to deploy previous version
git checkout <commit-hash>
git pull origin main --rebase
```

