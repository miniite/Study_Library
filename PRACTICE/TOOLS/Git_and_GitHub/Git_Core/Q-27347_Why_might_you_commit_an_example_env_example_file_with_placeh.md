---
type: "practice"
difficulty: "Basic"
---

# Why might you commit an example .env.example file with placeholder keys instead of the real .env in your LLM repo?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Gitignore](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitignore.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working on a repository that includes environment variables (like those used by Language Model (LLM) applications), it's common practice to commit an example .env.example file with placeholder keys rather than the actual .env file containing real credentials. This approach ensures security and provides flexibility for different environments without exposing sensitive information.

#### Triage

The question is probing why you might prefer using a template .env.example file over a fully populated .env file in your LLM repository. The key assumptions include:

The application requires environment variables to function correctly.
Security of credentials is paramount.
Different environments (development, staging, production) may have varying configurations.

#### Solution
from First Principles

Security: Committing sensitive information like API keys or database passwords directly in the source code can lead to security breaches if a repository is ever compromised. By using placeholders in .env.example, you maintain control over these secrets, which should be managed externally (e.g., environment-specific secret management services).

Flexibility and Reusability: The example file serves as a template that developers can easily understand and replicate for their specific environments. This makes it easier to onboard new team members or start new projects.

Version Control Best Practices: Sticking with .env.example files aligns better with version control best practices, which discourage committing sensitive data directly into the repository. External tools like dotenv or cloud-based secrets management can be used to manage and inject actual values at runtime.

Deployment Consistency: When deploying applications across different environments (development, staging, production), using a consistent template ensures that all necessary environment variables are set correctly without hardcoding them in the source code.

#### Closing Summary

In summary, committing a .env.example file is a safer and more scalable practice than including real credentials directly. It promotes better security practices, maintains version control integrity, and provides flexibility for different environments. Common mistakes include ignoring the importance of external secret management or not providing clear instructions on how to replace placeholder keys with actual values.

#### Common Mistakes/Follow-Ups
Overlooking External Secret Management: Developers sometimes overlook the need for tools like AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault.
Incomplete Instructions in README: Sometimes, the .env.example file lacks clear instructions on how to replace placeholder keys with actual values.
Hardcoding Sensitive Data Directly: Committing sensitive data directly into the source code, which can lead to security risks and compliance issues.

**Code:**

```bash
# Example .env.example content
API_KEY=your_api_key_here
DATABASE_URL=postgresql://user:password@localhost/dbname
```

**Flow:**

1. Repository Commit
2. Commit .env.example
3. Read .env.example and replaceplaceholders
4. Deploy to Environment
5. Use environment-specificsecrets management service

