---
type: "practice"
difficulty: "Intermediate"
---

# Why might you keep a separate branch for a risky rewrite of a slow Spark job while keeping the stable version on main?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Branching Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branching_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with large-scale data processing jobs like Spark jobs, it's crucial to maintain stability and reliability, especially in production environments where downtime can be costly. This is why you might choose to keep a separate branch for risky rewrites while keeping the stable version on main. Here’s how we can approach this:

Quick Overview: When rewriting a slow Spark job, we want to ensure that any changes don't introduce instability or bugs. Keeping a new, experimental version in a separate branch allows us to test thoroughly without affecting the main codebase.

#### Triaging the Question

What is being asked: The core of this question revolves around understanding when and why it makes sense to maintain a separate branch for risky changes.
What the interviewer is probing: They are testing your understanding of version control best practices, especially in complex projects where stability and risk management are critical.

#### First-Principles Solution

#### Assumptions
We assume you have a significant amount of data processing done by a Spark job that runs on main branch.
Steps to solve the problem:
Identify the Issue: Recognize that a Spark job is running slow, and there's an opportunity to rewrite it for better performance or new features.
Create a New Branch: Create a separate branch (e.g., feature/rewrite-spark-job) to implement the changes.
Implement Changes: Develop and test the new version of the Spark job in this isolated environment.
Test Extensively: Perform thorough testing, including unit tests, integration tests, and end-to-end tests to ensure no bugs are introduced.
Merge with Caution: Once thoroughly tested and proven stable, merge the changes into main with a detailed pull request that outlines all changes and their benefits.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Keeping the stable version on main ensures production remains unaffected during development.
This approach helps in systematically managing risks and ensuring smooth operations.

#### Common mistakes
include merging experimental features directly into main without proper testing, which can introduce bugs or performance issues.

