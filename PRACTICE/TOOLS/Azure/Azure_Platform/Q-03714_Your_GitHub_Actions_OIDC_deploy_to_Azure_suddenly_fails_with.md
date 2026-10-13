---
type: "practice"
difficulty: "Deep"
---

# Your GitHub Actions OIDC deploy to Azure suddenly fails with AADSTS70021; how do you debug the federated credential subject claim and trust configuration?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure DevOps and Artifact Governance](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we face a common challenge in DevOps where an automated deployment via GitHub Actions fails due to an authentication issue with Azure Active Directory (AAD). The specific error "AADSTS70021" suggests that there's a problem with the subject claim or trust configuration. To debug and resolve this, we need to carefully examine the OIDC (OpenID Connect) setup, ensuring both the GitHub Actions pipeline and Azure are properly configured.

The key idea is to systematically verify the federated credential subject claim and trust configuration on both sides: GitHub Actions and Azure. We will start by understanding what these claims represent and then proceed with a structured approach to identify any misconfigurations or missing steps.

**Code:**

```
MERMAID
Copy
graph TD;
    A[GitHub Actions OIDC] --> B[Azure AD App Registration];
    C[Azure App Service Authentication/Authorization] --> D[AADSTS70021 Error];
```

**Flow:**

1. Verify OIDC Config on GitHubActions
2. Check Azure AD AppRegistration
3. Configure Trust in Azure AppService
4. Test Deployment with NewConfigurations

