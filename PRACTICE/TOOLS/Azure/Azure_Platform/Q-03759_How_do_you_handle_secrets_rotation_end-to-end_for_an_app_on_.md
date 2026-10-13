---
type: "practice"
difficulty: "Deep"
---

# How do you handle secrets rotation end-to-end for an app on App Service pulling from Key Vault, so a rotated secret is picked up without a restart?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Key Vault and Secrets](<../../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure seamless handling of secret rotations in Azure for an application running on App Service that pulls secrets from Key Vault, we need to focus on several key areas: understanding how secrets are managed and rotated within Key Vault, integrating with Azure App Services, and ensuring no downtime during the rotation process. This involves setting up a Key Vault Access Policy, using Managed Identity or certificates for secure access, implementing a rotation strategy, and deploying appropriate monitoring and logging.

The main challenge is to ensure that when a secret is updated in Key Vault, the App Service picks up the new value without requiring a restart. This requires a deep understanding of Azure's built-in features such as Managed Identities and application configurations.

**Comparison:**

```
Secret Name	Current Value	New Value (After Rotation)
DBPassword	P@ssw0rd123	P@ssw0rd456
```

**Code:**

```csharp
// Example C# code snippet for accessing secrets from Key Vault using Managed Identity
using Microsoft.Azure.KeyVault;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Net.Http;

public class SecretManager
{
    private readonly string _keyVaultUri;
    private readonly string _clientId;

    public SecretManager(string keyVaultUri, string clientId)
    {
        _keyVaultUri = keyVaultUri;
        _clientId = clientId;
    }

    public async Task<string> GetSecretAsync(string secretName)
    {
        var client = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(GetToken));
        return await client.GetSecretAsync($"{_keyVaultUri}/{secretName}").GetAwaiter().GetResult();
    }

    private async System.Threading.Tasks.Task<AuthenticationResult> GetToken(string authority, string resource, string scope)
    {
        var context = new AdalAuthenticationContext(authority);
        var result = await context.AcquireTokenAsync(resource, _clientId);
        return result;
    }
}
```

**Example input / output:**

```text
Input:
Key Vault URI: https://mykeyvault.vault.azure.net
Client ID: 00000000-1111-2222-3333-444444444444

Output:
Secret retrieved from Key Vault successfully.
```

**Data:**

```json
{
  "KeyVaultUri": "https://mykeyvault.vault.azure.net",
  "ClientId": "00000000-1111-2222-3333-444444444444"
}
```

**Flow:**

1. No
2. Yes
3. App Service
4. Requests Secret from Key Vault
5. Key Vault Access Policy
6. Managed Identity or Certificate
7. Is Secret Updated?
8. App Service Uses New Value
9. Trigger Rotation in Key Vault
10. Key Vault Rotates Secret
11. Notification Sent to AppService
12. App Service Refreshes Secrets

