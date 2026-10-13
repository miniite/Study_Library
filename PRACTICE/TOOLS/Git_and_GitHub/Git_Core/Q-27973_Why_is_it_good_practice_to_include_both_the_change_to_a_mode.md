---
type: "practice"
difficulty: "Basic"
---

# Why is it good practice to include both the change to a model and its updated data-quality test in the same commit so a reviewer sees the logic and the check together?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conventional Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Conventional_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, keeping changes to a model and its corresponding data-quality tests within the same commit is a best practice because it ensures that the logic applied to the data and the validation of that logic are aligned and transparent. This practice helps maintain code integrity and facilitates easier review processes by reviewers.

To understand why this is beneficial, we need to consider the following:

Code Integrity: Changes in data processing models should be tested immediately after they are made to ensure that the new logic works as intended.
Transparency and Accountability: Keeping related changes together provides a clear audit trail of what was modified and how it affects the data quality.

Let's delve into why this is important:

Logical Consistency: When you update a model, any data-quality test associated with that change should be tested immediately to ensure both are in sync. This practice ensures that the logic applied to the data is thoroughly validated.
Reproducibility: By committing changes together, it becomes easier for others (or yourself) to understand and reproduce the changes later if needed.

Now, let's step through how this works from first principles:

Change to Model: Suppose you have a function that processes raw data into cleaned data. This change might involve adding new features or modifying existing ones.
Data-Quality Test: After making such changes, it is essential to update the tests to reflect these new functionalities. For instance, if you added a new feature to clean dates, you should also write and commit a test that verifies this functionality.

To illustrate with an example:

Initial Code:
PYTHON
def clean_dates(dates):
    return [date for date in dates if date > datetime.now()]

# Test
assert all(clean_dates([datetime(2023,1,1), datetime.now()]) == [datetime(2023,1,1)])
After Change:
PYTHON
def clean_dates(dates):
    return [date for date in dates if date >= datetime.now()]

# Test
assert all(clean_dates([datetime(2023,1,1), datetime.now()]) == [datetime(2023,1,1), datetime.now()])

By keeping these changes together, the reviewer can easily see that the logic in clean_dates has been updated and that the test reflects this change.

