---
type: "practice"
difficulty: "Deep"
---

# Give an external annotator team push access to exactly one eval-dataset repo and one branch without granting broad org access to your models code. Compare deploy keys, fine-grained PATs, and outside-collaborator roles.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ OIDC Federation for Deploys](<../../../../TOOLS/Git_and_GitHub/Git_Core/OIDC_Federation_for_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To provide an external annotator team push access to exactly one eval-dataset repo while ensuring they do not gain broad org access to your models' code, we need to carefully consider and compare three methods: deploy keys, fine-grained Personal Access Tokens (PATs), and outside-collaborator roles in GitHub. Each method has its pros and cons, which we will evaluate from the perspective of security, manageability, and ease of use.

