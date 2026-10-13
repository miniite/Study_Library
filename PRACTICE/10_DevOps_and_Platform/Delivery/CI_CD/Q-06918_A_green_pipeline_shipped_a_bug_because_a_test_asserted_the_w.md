---
type: "practice"
difficulty: "Deep"
---

# A green pipeline shipped a bug because a test asserted the wrong thing; how do you build guardrails (mutation tests, PR templates, review focus) so tests actually catch regressions?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Integration Testing and Test Doubles](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Integration_Testing_and_Test_Doubles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

In this scenario, we need to enhance our CI/CD pipeline by implementing several guardrails such as mutation testing and robust pull request (PR) templates to ensure that tests effectively catch regressions. The key is to focus on code quality, maintainability, and developer education.

#### Triaging the Question

The core issue here is a regression in a green pipeline due to a flawed test assertion. The interviewer is probing our understanding of CI/CD best practices, specifically how we can build robust guardrails to prevent such issues. We need to consider static code analysis tools, mutation testing, PR templates, and review processes.

#### Assumptions
Our project uses modern development practices.
We have a well-established CI/CD pipeline with multiple stages.
The current issue is isolated to a specific test failing the assertion rather than a broader build or deployment failure.

#### First Principles Solution

Mutation Testing: Mutation testing involves intentionally introducing small changes (mutations) into the code and running tests to see if they are caught by existing tests. This helps ensure that our tests cover all critical paths.

Static Code Analysis Tools: Utilize tools like SonarQube, ESLint, or Pylint to identify potential issues before they reach the test phase.

Pull Request Templates: Create a PR template that requires developers to document changes, including any new tests added and why certain assertions were changed. This ensures transparency and accountability.

Code Reviews: Implement strict code review processes where reviewers focus on testing strategies and whether tests adequately cover the codebase.

Test Coverage Reporting: Regularly report test coverage metrics to developers, highlighting areas with low coverage that need improvement.

Automated Tests for Business Logic: Ensure comprehensive unit tests and integration tests cover business logic thoroughly. Use frameworks like Jest or PyTest if needed.

Environment Setup and Isolation: Make sure the development environment closely mirrors the production environment to catch issues early.

Detailed Implementation:

Mutation Testing Example (Using Jester in a Node.js Project):

JAVASCRIPT
// Before Mutation Testing

#### cons
t add = (a, b) => {
  return a + b;
};

test('adds 1 + 2 to equal 3', () => {
  expect(add(1, 2)).toBe(4); // This will fail the mutation testing.
});

// After Mutation Testing

#### cons
t add = (a, b) => {
  return a + b;
};

test('adds 1 + 2 to equal 3', () => {
  expect(add(1, 2)).toBe(3); // This will pass the mutation testing.
});

PR Template Example:

MARKDOWN
# Pull Request Template

## Description
- [ ] Explain the changes made in a clear and concise manner.
- [ ] Identify any related issues or pull requests.

## Testing
- [ ] Added new unit tests for the following scenarios:
  - [ ] Test 1
  - [ ] Test 2
- [ ] Verified that existing tests still pass.
- [ ] Documented why certain assertions were changed in `test_description.md`.

## Additional Notes
- [ ] Any additional notes or observations about the PR.

Code Review Example: During a code review, focus on:

MARKDOWN
- Do tests cover all critical paths?
- Are there any edge cases missing in the tests?
- Does the new test add meaningful value to the existing suite?

#### Closing Summary
and Common Mistakes/Follow-Ups:

#### Closing Summary
Implementing mutation testing, using static code analysis tools, creating robust PR templates, and conducting thorough code reviews are essential steps to prevent regressions in your CI/CD pipeline. By focusing on these guardrails, you can ensure that tests are not only written but also effective at catching issues.

#### Common Mistakes/Follow-Ups

Ignoring False Positives: Ensure that test failures are truly indicative of issues and not just noise.
Overlooking Code Reviews: Regular code reviews help catch issues early and maintain high code quality standards.
Insufficient Test Coverage: Always strive to increase test coverage, especially for critical paths.

Would you like to discuss any specific tool or approach further? For example, how mutation testing can be integrated with our existing CI/CD pipeline?

**Comparison:**

```
Step	Action
1	Implement mutation testing.
2	Use static code analysis tools.
3	Create PR templates for documentation.
4	Enforce strict code reviews.
```

**Code:**

```javascript
// Mutation Testing Example
const add = (a, b) => {
  return a + b;
};

test('adds 1 + 2 to equal 3', () => {
  expect(add(1, 2)).toBe(4); // This will fail the mutation testing.
});
```

**Example input / output:**

```text
Input:
- New feature added with incomplete test coverage.
Output:
- PR template filled out, including new tests and documentation of changes.
- Code reviewed and merged after addressing feedback.
```

**Data:**

```json
{
  "pr_template": {
    "description": "Explain the changes made in a clear and concise manner.",
    "testing": [
      "Added new unit tests for the following scenarios:",
      "- Test 1",
      "- Test 2",
      "Verified that existing tests still pass."
    ]
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Start
6. Is Mutation Testing in Place?
7. Implement Mutation Testing
8. Run Static Code Analysis
9. Do PR Templates Exist?
10. Use PR Templates
11. Create PR Templates
12. Conduct Code Reviews
13. Report Test Coverage Metrics
14. Run Tests and Merge

