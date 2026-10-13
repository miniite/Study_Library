---
type: "practice"
difficulty: "Intermediate"
---

# During code review you see a PR mixing a log-parser refactor, a new alert rule, and a metric-relabel fix in one commit. Explain concretely why this hurts revertability and bisect for a production monitoring service, and how you'd coach the author to restructure it.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Atomic Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Atomic_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a code review that includes multiple unrelated changes in one commit—such as a log-parser refactor, a new alert rule, and a metric-relabel fix—it's important to consider several factors that affect maintainability, revertibility, and bisecting abilities. This PR is a good candidate for refactoring into smaller, more focused commits.

The core issue with the current approach is that it makes it difficult to understand the impact of individual changes, especially in complex systems like production monitoring services where multiple components interact closely. Here’s how this can hurt revertibility and bisecting:

Revertibility: If a single commit contains multiple unrelated changes, reverting to a previous version becomes risky. For instance, if only one change is causing issues, it might be challenging to pinpoint the exact problematic change without fully understanding all three modifications.

Bisecting: Bisecting helps in finding the exact commit that introduced a bug or regression by iteratively halving the codebase changes between two known states (good and bad). With multiple unrelated changes mixed together, bisecting becomes much harder because any intermediate state might not be functional due to the interdependence of the changes.

To coach the author on better structuring the PR, we can follow these steps:

Identify Independent Changes: First, identify each change independently. For example:

Change 1: Log-parser refactor.
Change 2: New alert rule implementation.
Change 3: Metric relabel fix.

Isolate Each Change: Create separate PRs for each identified change. This ensures that each PR is focused on a single, specific task and makes it easier to understand the impact of individual changes.

Test Independently: Ensure that each PR undergoes thorough testing before merging. This helps in identifying any regressions or issues early on, making them easier to address.

Document Changes: Document why each change was made and link relevant issue tickets. This makes it clear what the purpose of each commit is and aids future maintenance.

Pair Programming or Code Reviews: Engage with the author for a code review session where you can walk through the changes together, ensuring that they are logically sound and well-documented.

By restructuring the PR into smaller, focused commits, we enhance the maintainability and reliability of the production monitoring service.

**Code:**

```
MARKDOWN
Copy
# Example of how to structure the PR

## Change 1: Log-parser refactor
- Refactor log parser to improve performance and readability.
- Add unit tests for new functions.
- Update documentation on logging.

## Change 2: New alert rule implementation
- Implement a new alert rule based on recent business requirements.
- Write comprehensive test cases covering various scenarios.
- Document the rationale behind the new rule.

## Change 3: Metric relabel fix
- Correctly relabel metrics to ensure accurate monitoring data.
- Update metric definitions and dependencies in the system.
- Add logging around critical paths for debugging purposes.
```

**Example input / output:**

```
Example Input and Output for a PR Review

Input:

CODE
Copy
PR Title: Introduce log-parser refactor, new alert rule, and metric relabel fix
Description:
- Refactor the log parser to improve performance.
- Implement a new alert rule based on recent requirements.
- Fix incorrect metric labels.

Output:

CODE
Copy
PR Title: Log-parser refactor

PR Description:
- Refactor the log parser to improve performance.
- Add unit tests for new functions.
- Update documentation on logging.

PR Title: New alert rule implementation

PR Description:
- Implement a new alert rule based on recent business requirements.
- Write comprehensive test cases covering various scenarios.
- Document the rationale behind the new rule.

PR Title: Metric relabel fix

PR Description:
- Correctly relabel metrics to ensure accurate monitoring data.
- Update metric definitions and dependencies in the system.
- Add logging around critical paths for debugging purposes.
```

**Data:**

```text
# Sample PR structure

| Change | Description |
|--------|-------------|
| Log-parser refactor | Refactor log parser, add tests, update docs. |
| New alert rule implementation | Implement new alert rules, write test cases, document rationale. |
| Metric relabel fix | Fix metric labels, update definitions, add debugging logs. |
```

