---
type: "practice"
difficulty: "Deep"
---

# A resource shows perpetual diff on every plan even with no code change. How do you debug ignore_changes, computed fields, and provider bugs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring with State Moves](<../../../../TOOLS/Terraform/Terraform_Core/Refactoring_with_State_Moves.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a resource in Terraform always shows a "perpetual diff" during planning, it means that Terraform is detecting changes even when there are none. This can be due to several factors including ignore_changes, computed fields, or issues within the provider itself. To debug these issues effectively, we need to systematically approach each potential cause.

**Code:**

```hcl
module "example" {
  source = "./modules/example"

  # Example of an ignore_changes configuration
  resource = "aws_instance.example"
  ignore_changes = ["tags"]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Identify the Issue
10. Is 'ignore_changes' improperlyconfigured?
11. Check `ignore_changes`configuration
12. Are there computed fieldscausing issues?
13. Review computed field logic
14. Is the provider at fault?
15. Report a bug to the provideror seek community help
16. Resource metadata issues?Check logs and state file
17. Are there any redundant fieldsbeing ignored?
18. Adjust `ignore_changes`configuration
19. Verify the resource's attributesare correctly handled byTerraform

