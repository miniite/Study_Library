---
type: "practice"
difficulty: "Deep"
---

# A prompt change was reviewed and merged, but no one noticed it removed a critical safety instruction; how would you make prompt diffs reviewable and gate-able like code diffs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pull Request Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that prompt diffs are reviewable and gate-able just like code diffs, we need to establish a robust CI/CD pipeline that integrates with our existing version control system (e.g., Git). The core idea is to treat prompts as first-class citizens in our development process, subjecting them to the same rigorous scrutiny as any other change. This approach ensures that safety instructions and other critical information are not inadvertently removed during updates.

#### Triage

The question is probing how we can integrate prompt diffs into a CI/CD pipeline to ensure they receive proper review and gating before being deployed. We need to make several assumptions, such as the availability of a version control system, access to a CI/CD toolchain, and existing knowledge about the current development workflow.

#### First-Principles Solution
Prompt Storage: Ensure prompts are stored in a version-controlled repository (e.g., Git) alongside other code.
Code Diff Integration: Integrate prompt diffs into the same diff tools used for code changes to highlight modifications.
Review Process: Modify existing pull request reviews to include checks on prompt diffs, ensuring that critical instructions and safety measures are not overlooked.
Gating Mechanisms: Use gating mechanisms (e.g., approval workflows) to ensure prompts pass review before deployment.
Step-by-Step Solution

Storage Strategy:

Store prompts in a dedicated directory within the repository, similar to how code is stored.

#### Example
./prompts/safety_instructions.txt

Diff Integration:

Integrate diff tools (e.g., Git) to show changes between versions of prompt files.
Use existing CI/CD pipelines to trigger diffs when a pull request is created.

Review Process:

Modify the existing PR review process to include prompts as part of the checklist.
Create a custom script or plugin for diff tools that highlights any changes in critical safety instructions.

Gating Mechanisms:

Implement automated checks within the CI/CD pipeline to ensure prompts pass before deployment.
Use conditional gate steps (e.g., manual approval) if critical changes are detected.

#### Closing Summary

By treating prompt diffs as first-class citizens, we can integrate them into our existing CI/CD pipelines and ensure they receive proper review and gating. This approach not only enhances the robustness of our system but also mitigates risks associated with removing critical safety instructions or other important information.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to store prompts in a version-controlled repository.
Mistake 2: Not integrating diff tools to highlight changes between versions.
Mistake 3: Skipping the review process for prompt diffs, leading to oversight.

#### Follow-Up
Discussing how to handle large or complex prompts and ensuring that the integration does not introduce performance bottlenecks.

**Comparison:**

| Step | Action | Description |
|------|--------|-------------|
| 1    | Store prompts in Git | Ensure prompts are version-controlled alongside code. |
| 2    | Integrate diff tools | Use existing diff tools to show changes between versions of prompt files. |
| 3    | Modify PR reviews | Include checks for critical safety instructions as part of the review process. |
| 4    | Implement gating mechanisms | Use automated checks and manual approvals to ensure prompts pass before deployment. |

**Code:**

```python
# Example of a script to highlight changes in safety instructions
import difflib

def diff_files(file1, file2):
    with open(file1) as f1:
        with open(file2) as f2:
            diff = difflib.unified_diff(f1.readlines(), f2.readlines())
            for line in diff:
                if "safety" in line:  # Example keyword
                    print(line)

# Usage
diff_files('prompts/safety_instructions.txt.orig', 'prompts/safety_instructions.txt')
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `diff_files('prompts/safety_instructions.txt.orig', 'prompts/safety_instructions.txt')` | Diff output highlighting lines containing the keyword "safety" between the two files. |

**Data:**

```json
{
  "file1": "prompts/safety_instructions.txt.orig",
  "file2": "prompts/safety_instructions.txt"
}
```

**Flow:**

1. Store Prompts in Git
2. Integrate Diff Tools
3. Modify PR Reviews
4. Implement Gating Mechanisms
5. Deploy Changes After Review

