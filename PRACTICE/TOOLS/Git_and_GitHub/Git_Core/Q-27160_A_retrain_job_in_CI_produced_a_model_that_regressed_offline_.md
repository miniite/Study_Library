---
type: "practice"
difficulty: "Deep"
---

# A retrain job in CI produced a model that regressed offline AUC by two points, and you suspect a silent change in a Git-tracked feature-store definition. Walk through using git log -L on the feature spec plus dvc diff to isolate whether it was the schema or the encoder.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Versioning Data with DVC](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Data_with_DVC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to methodically trace the changes in our Git-tracked feature store definition and data versioning using tools like git log -L and DVC (Data Version Control) diff. The goal is to isolate whether a change in the schema or encoder of the feature spec is causing the offline AUC regression.

#### Triage
Understanding the Problem
Problem Statement: We have an ML model that has regressed by two points in offline AUC post-retraining, suggesting a potential issue with the data.
Tools and Context:
Git for version control of the feature store definition.
DVC to manage and track changes in dataset and models.

#### First-Principles Solution
Identify the Feature Spec Change:
Use git log -L to find the exact lines that were modified in the feature spec file.
Analyze Changes with DVC Diff:
Run dvc diff to see how the data and model have changed between commits.
Step-by-Step Solution

Identify the Commit Range:

First, identify the commit range where the retraining occurred using Git log.
BASH
git log --oneline --after="2023-10-01" --before="2023-10-05"
Assume we found a relevant commit commit-hash around this time.

Use git log -L:

Use git log -L<start>,<end>:<file> to pinpoint the exact lines changed in the feature spec file.
BASH
git log -L 10,20:features/config.yaml -- <commit-hash>
This command will show changes between line 10 and line 20 of features/config.yaml in commit commit-hash.

Analyze the Changes:

Inspect the output from git log -L. It will highlight the exact lines that were changed.
DIFF
@@ -10,8 +10,6 @@
 feature-a: 1
 #+feature-b: 2
 -feature-c: 3
 -feature-d: 4
 feature-e: 5

Check with DVC Diff:

Run dvc diff to see if there were any changes in the dataset or model configuration.
BASH
dvc diff <commit-hash>
This will provide a detailed breakdown of what changed, including feature encoders and data transformations.

Dry-Run the Changes:

If necessary, create a new branch to test these changes with a fresh retrain job.
BASH
git checkout -b test-feature-change <commit-hash>
dvc repro --force

#### Closing Summary

By methodically using git log -L and DVC diff, we can identify the exact lines that were changed in the feature spec and determine if these changes are causing the AUC regression. This process ensures a systematic approach to troubleshooting data-related issues.

#### Common Mistakes/Follow-Ups
Failing to Isolate Changes: Not using git log -L precisely enough may lead to missing relevant changes.
Overlooking DVC Diff Outputs: Focusing only on Git logs might miss important context from DVC diffs, such as data transformations.
Not Testing with New Branches: Skipping a dry-run in a new branch can make it harder to diagnose and fix the issue.

**Comparison:**

| Feature | Old Value | New Value |
|---------|-----------|-----------|
| feature-c | 3         | -         |
| feature-d | 4         | -         |

**Code:**

```bash
git log --oneline --after="2023-10-01" --before="2023-10-05"
git log -L 10,20:features/config.yaml -- <commit-hash>
dvc diff <commit-hash>
```

**Example input / output:**

```

Input: git log --oneline --after="2023-10-01" --before="2023-10-05"

Output:

CODE
Copy
commit-hash (commit message)

Input: git log -L 10,20:features/config.yaml -- <commit-hash>

Output:

DIFF
Copy
@@ -10,8 +10,6 @@
 feature-a: 1
 #+feature-b: 2
 -feature-c: 3
 -feature-d: 4
 feature-e: 5

Input: dvc diff <commit-hash>

Output:

DIFF
Copy
Data changes:
  features/train.csv (modified)
Model changes:
  model.pkl (retrained)
```

**Data:**

```yaml
# Example features/config.yaml
feature-a: 1
+feature-b: 2
-feature-c: 3
-feature-d: 4
feature-e: 5
```

**Flow:**

1. Yes
2. Identify Commit Range
3. Use `git log` to find relevantcommit
4. Run 'dvc diff <commit-hash>'
5. Analyze changes with `dvc diff`and `git log -L`
6. Test with a new branch ifnecessary
7. Review model performance ontest data

