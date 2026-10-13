---
type: "practice"
difficulty: "Intermediate"
---

# Explain the risk of a module reading environment variables or local files directly. Why does that hurt reproducibility and state consistency across machines?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Module Compatibility](<../../../../TOOLS/Terraform/Terraform_Core/Module_Compatibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using Terraform to manage infrastructure as code, it's essential to ensure that your configurations are consistent, repeatable, and easily manageable. One critical aspect of this is managing environment variables and local files directly within modules. Let’s break down the risks associated with this practice.

Firstly, let's clarify what we're dealing with: Modules in Terraform are reusable code blocks that can be defined once and reused multiple times across different configurations. However, when a module reads environment variables or local files directly, it introduces several potential issues related to reproducibility and state consistency.

**Code:**

```
TERRAFORM
Copy
module "example_module" {
  source = "./modules/example"

  # Directly reading environment variable
  env_var_value = var.env_var

  # Directly reading a local file
  local_file_path = file("${path.module}/local-file.txt")
}
```

