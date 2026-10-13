---
type: "practice"
difficulty: "Intermediate"
---

# How do you enforce secret rotation for a legacy app that can't re-read its config without a restart?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shared Secret Rotation](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Shared_Secret_Rotation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce secret rotation for a legacy application that cannot reload its configuration without a restart, we need to ensure that the secrets are securely stored and rotated on a schedule, while minimizing downtime during the rotation process. This involves designing a mechanism that can notify the application of the new secrets without requiring it to be restarted. We will explore using Hashicorp Vault for secret management and discuss how to integrate this with our legacy app.

#### Triage

The question is asking how to handle secret rotation in an environment where the application does not support configuration reloads, meaning we need a way to notify the application of new secrets without requiring it to restart. The key areas of focus are:

Ensuring secure storage and management of secrets using Hashicorp Vault.
Implementing a mechanism for notifying the application about secret changes.
Minimizing downtime during the rotation process.

#### First-Principles Solution
Secret Storage: Store sensitive information, such as API keys or database credentials, in Hashicorp Vault. Vault provides strong security features and can be configured to rotate secrets automatically.
Rotation Mechanism: Create a script or service that periodically checks for updated secrets in Vault. When new secrets are available, this script should update the application's configuration file or environment variables.
Application Notification: Develop an interface (e.g., HTTP endpoint) within the legacy app to accept updates from the rotation mechanism.
Rolling Update Strategy: Implement a rolling update strategy to minimize downtime and ensure high availability during secret rotations.
Step-by-Step Solution

Vault Configuration:

Set up Hashicorp Vault with appropriate policies and access controls.
Enable automatic secret rotation for sensitive data, e.g., using ttl and max_ttl policies.

Rotation Script:

BASH
#!/bin/bash

VAULT_ADDR="https://vault.example.com"
VAULT_TOKEN="your-vault-token"

SECRET_PATH="/secret/app-creds"
CONFIG_FILE="/path/to/application/config.ini"

# Fetch the current secret version from Vault
CURRENT_SECRET=$(vault read -field=value $SECRET_PATH | jq -r .data.value)

# Check if the secret has changed since the last update
if [ "$(cat $CONFIG_FILE)" != "$CURRENT_SECRET" ]; then
  echo "Secrets have been updated. Updating application configuration..."

  # Update the config file with new secrets
  sed -i "s/^SECRET_KEY=.*/SECRET_KEY=$CURRENT_SECRET/" $CONFIG_FILE

  # Notify the application about the update (e.g., via HTTP)
  curl -X POST http://localhost:8080/reload-config
fi

Application Code: Modify the legacy app to accept a configuration reload request and apply the new secrets.

PYTHON
import os
import requests

def reload_config():
    config_file = '/path/to/application/config.ini'
    try:
        with open(config_file, 'r') as f:
            lines = f.readlines()
        for i, line in enumerate(lines):
            if "SECRET_KEY" in line:
                lines[i] = 'SECRET_KEY=' + os.environ.get('NEW_SECRET', '') + '\n'
        with open(config_file, 'w') as f:
            f.writelines(lines)
    except Exception as e:
        print(f"Failed to reload config: {e}")

if __name__ == '__main__':
    while True:
        response = requests.post('http://localhost:8080/reload-config')
        if response.status_code == 200:
            reload_config()
            break

Rolling Update Strategy:

Schedule the rotation script to run during a low-traffic period.
Monitor the application’s health and performance after each update.
Implement load balancing and failover mechanisms if necessary.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we have implemented a robust solution by leveraging Hashicorp Vault for secret management and creating a notification mechanism to update the legacy app without requiring a restart. Key considerations include secure secret storage, efficient rotation scripts, and minimizing application downtime. Common mistakes include not properly configuring Vault policies, neglecting to handle potential race conditions during updates, and failing to monitor the system post-update.

For further discussion, we could explore advanced features like automated rolling updates using orchestration tools or integrating with a CI/CD pipeline for seamless secret management.

**Comparison:**

```
Step	Action
1	Set up Hashicorp Vault for secret management.
2	Create a script to fetch updated secrets from Vault and notify the app.
3	Modify the legacy application to accept configuration reloads.
4	Schedule periodic rotation checks during low-traffic periods.
```

**Code:**

```

#!/bin/bash

VAULT_ADDR="https://vault.example.com" VAULT_TOKEN="your-vault-token"

SECRET_PATH="/secret/app-creds" CONFIG_FILE="/path/to/application/config.ini"

CURRENT_SECRET=$(vault read -field=value $SECRET_PATH | jq -r .data.value)

if [ "$(cat $CONFIG_FILE)" != "$CURRENT_SECRET" ]; then echo "Secrets have been updated. Updating application configuration..."

sed -i "s/^SECRET_KEY=.*/SECRET_KEY=$CURRENT_SECRET/" $CONFIG_FILE

curl -X POST http://localhost:8080/reload-config fi

CODE
Copy

```python
import os
import requests

def reload_config():
    config_file = '/path/to/application/config.ini'
    try:
        with open(config_file, 'r') as f:
            lines = f.readlines()
        for i, line in enumerate(lines):
            if "SECRET_KEY" in line:
                lines[i] = 'SECRET_KEY=' + os.environ.get('NEW_SECRET', '') + '\n'
        with open(config_file, 'w') as f:
            f.writelines(lines)
    except Exception as e:
        print(f"Failed to reload config: {e}")

if __name__ == '__main__':
    while True:
        response = requests.post('http://localhost:8080/reload-config')
        if response.status_code == 200:
            reload_config()
            break
```

**Example input / output:**

```text
Input (rotation script):
1. Fetch current secret from Vault.
2. Compare with local config file.
3. Update config if different.

Output (application code):
1. Accept update request via HTTP endpoint.
2. Reload application configuration based on new secrets.
```

**Data:**

```json
{
  "vault": {
    "addr": "https://vault.example.com",
    "token": "your-vault-token"
  },
  "config_file_path": "/path/to/application/config.ini",
  "secret_path": "/secret/app-creds"
}
```

**Flow:**

1. Yes
2. No
3. Check for secret updates
4. Is there a new secret?
5. Update application config
6. Notify app to reload config
7. Wait until next check
8. Monitor and validate update

