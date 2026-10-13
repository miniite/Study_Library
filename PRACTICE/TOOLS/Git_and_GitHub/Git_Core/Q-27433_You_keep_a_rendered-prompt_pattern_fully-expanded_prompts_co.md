---
type: "practice"
difficulty: "Deep"
---

# You keep a rendered-prompt pattern (fully-expanded prompts committed) versus templates-in-Git. Compare the two for reviewability, drift detection against production, and rollback of a GenAI change.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Commit Messages](<../../../../TOOLS/Git_and_GitHub/Git_Core/Commit_Messages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When comparing the "rendered-prompt" pattern with templates in Git for AI engineering, it's important to consider the aspects of reviewability, drift detection against production, and the ability to rollback a GenAI change. The rendered-prompt approach fully expands the prompt before committing, whereas templates are stored as code and only expanded at runtime.

The key difference lies in how these approaches handle complexity and maintain consistency:

Rendered-Prompt Pattern: This pattern commits the final, expanded form of the prompt directly to Git. It ensures that every commit includes a complete, executable version of the prompt.
Templates in Git: Templates are stored as code and expanded at runtime. This approach keeps the code clean and separate from its rendered output.

#### Triage

The question is probing your understanding of Git workflows, how GenAI systems work, and best practices for maintaining consistent and auditable codebases. We need to assume that we have a production environment with deployed AI models, and our goal is to ensure that changes made in the development process are accurately reflected in the deployment.

#### First-Principles Solution
Reviewability
Rendered-Prompt Pattern:
Each commit includes the fully expanded prompt.
Changes are immediately visible and understandable from the Git history.
Templates in Git:
The rendered output is not directly visible in the repository.
Developers must ensure that template logic is correct, which can be error-prone if the templates are complex.
Drift Detection Against Production
Rendered-Prompt Pattern:
Direct comparison of commits with production ensures consistency.
Any drift between development and production prompts is immediately visible in Git diffs.
Templates in Git:
Continuous integration (CI) pipelines must monitor template logic to detect any changes that could introduce drift.
This requires additional complexity in CI/CD processes.
Rollback of a GenAI Change
Rendered-Prompt Pattern:
Rolling back involves reverting the commit and redeploying the exact previous version.
Simple, direct, and reliable.
Templates in Git:
Rolling back requires re-applying old templates to generate new prompts.
This process might involve complex logic if template dependencies have changed over time.

#### Closing Summary

The choice between rendered-prompt patterns and templates in Git depends on the specific requirements of your project. Rendered prompts offer simpler review processes, easier drift detection, and straightforward rollback mechanisms. However, they can become unwieldy with overly complex prompts. Templates provide cleaner code but require additional effort to manage template logic correctly.

#### Common Mistakes/Follow-Ups
Assuming templates are always better: This overlooks the complexity that comes with maintaining dynamic template systems.
Over-relying on rendered prompts without proper CI/CD checks: This can lead to hidden drift issues.
Failing to automate rollbacks effectively: Automated scripts should be in place regardless of which approach you choose.

**Comparison:**

```
Approach	Reviewability	Drift Detection	Rollback
Rendered Prompt Pattern	Direct and simple	Immediate visibility	Simple, direct
Templates in Git	Complex CI/CD	Requires monitoring	More complex logic
```

**Code:**

```python
# Example of a simple rendered prompt (rendered-prompt pattern)
prompt = "Generate a short story about a {character} who {adventure}."
full_prompt = render_prompt(prompt, character="wizard", adventure="journey to the magic land")

# Template in Git example
def generate_prompt(character, adventure):
    return f"Generate a short story about a {character} who {adventure}."
```

**Example input / output:**

```

Rendered Prompt Example:

Input (Git commit message): Updated prompt with new adventure story.
Output (Commit):
CODE
Copy
Generate a short story about a wizard who journey to the magic land.

Template Example:

Input (Git commit message): Updated template for adventures.
Output (Generated Prompt):
PYTHON
Copy
def generate_prompt(character, adventure):
    return f"Generate a short story about a {character} who {adventure}."
```

**Data:**

```json
{
  "prompt": "Generate a short story about a wizard who journey to the magic land."
}
```

**Flow:**

1. Rendered-Prompt
2. Commit rendered prompt
3. Rollback to previous commit
4. Direct comparison withproduction
5. Template in Git
6. Check template logic in CI/CD
7. Rollback involves re-applyingold templates

