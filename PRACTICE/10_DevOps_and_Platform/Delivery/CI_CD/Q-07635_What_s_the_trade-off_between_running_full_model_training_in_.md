---
type: "practice"
difficulty: "Deep"
---

# What's the trade-off between running full model training in CI on every PR versus a smaller smoke-train on a data subset?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Integration Testing and Test Doubles](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Integration_Testing_and_Test_Doubles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In continuous integration (CI) systems, deciding whether to run a full model training pipeline or a smaller "smoke-train" on a subset of data is crucial. Running a full train on every pull request (PR) can ensure the latest code changes don't break anything but comes with significant overhead in terms of time and computational resources. Conversely, using a smoke-test with a small dataset can provide quick feedback while significantly reducing resource usage, but it risks not catching all issues.

#### Triage

The core question here is about understanding the trade-offs between two strategies: running a full model training pipeline versus a smaller "smoke-train" on a subset of data. The interviewer is likely probing your knowledge of CI/CD best practices, your ability to balance accuracy with efficiency, and how you would evaluate the effectiveness of different approaches.

#### First-Principles Solution

Full Train on Every PR:

#### Pros
Ensures that any code changes don't break existing functionality.
Provides a comprehensive test of the model's performance across all data.

#### Cons
High resource consumption, which can lead to longer CI pipeline times and higher costs.
May delay feedback on new features or bug fixes.

Smoke-Train on Subset Data:

#### Pros
Reduces the time spent waiting for the full train to complete, providing quicker feedback on changes.
Lower resource consumption, which can help in managing CI pipeline costs and reducing delays.

#### Cons
May not catch all issues that could arise from model training on the entire dataset.
Risks introducing bugs or performance degradation if the subset is not representative of the full dataset.
Evaluating the Trade-offs

To decide between these two approaches, consider the following:

Representativeness of Data: Ensure that the subset used for smoke-testing is representative of the overall dataset. This can be achieved by stratified sampling techniques.
Performance Metrics: Define key performance metrics (KPM) such as accuracy, loss, and other relevant statistics to evaluate both full train and smoke-train results.
CI Pipeline Timeouts: Set appropriate timeouts to balance between waiting for the full train to complete and the urgency of getting feedback on new changes.
Error Detection Rate: Measure how often smoke-tests fail to catch issues that are only visible in a full train. This can help in optimizing the size and composition of the subset used.

#### Closing Summary

In summary, running full model training on every PR is more thorough but resource-intensive, whereas using a smaller "smoke-train" provides quicker feedback at lower cost. The choice depends on the trade-off between ensuring comprehensive testing versus minimizing CI pipeline times. A combination of both approaches can be effective: use smoke-tests for quick iterations and full trains to catch more complex issues.

#### Common Mistakes
and Follow-Ups
Ignoring Data Representativeness: Ensuring that the subset data is representative is crucial; otherwise, smoke-tests might miss significant issues.
Over-reliance on Smoke-Tests: Relying too heavily on smaller datasets can lead to false confidence in the model's performance.
Neglecting Performance Metrics: Focusing only on the speed of CI pipelines without evaluating the actual impact on model quality.

#### Follow-ups
could include discussing how to automate the process, integrating with other tools like Linting and static code analysis, or exploring hybrid approaches that balance these factors effectively.

