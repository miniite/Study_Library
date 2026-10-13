---
type: "practice"
difficulty: "Deep"
---

# Your Python service exposes an internal admin endpoint that triggers a config reload. How would you make the reload atomic, validate the new config before swapping it in, and roll back if the new config is bad?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Safe User-authored Configuration](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our Python service can safely handle a config reload via an admin endpoint while maintaining atomicity, validating the new configuration, and rolling back on failure, we need to implement a series of steps that cover both code-level and operational aspects.

The core idea is to use a combination of file locking, temporary directories, and a validation function to ensure that only valid configurations are swapped in. We'll break this down step by step:

Understanding the Requirements: The service should be able to accept an admin request to reload its configuration, validate the new config, and either swap it in or revert back to the previous configuration if the validation fails.

File Locking and Temporary Directories:

Use a file lock mechanism to prevent multiple reloads from happening simultaneously.
Create a temporary directory for the new configuration files before swapping them with the live ones.

Validation Function: Develop a function that checks the new config against known good formats or business rules. This could involve parsing the JSON/YAML and ensuring it adheres to expected schemas.

Swapping Configuration:

Once validated, swap in the new configuration files.
If validation fails, revert back to the old configuration without any service downtime.

Error Handling and Logging: Implement comprehensive error handling and logging to capture issues during the reload process.

Let's dive into a practical implementation of this approach.

**Comparison:**

```
Step	Action
1	Acquire lock on config file
2	Validate new config files
3	Create a temporary directory for new config files
4	Copy old config files to temp dir
5	Replace old config with new ones
6	Rename temp dir to current config dir
7	Rollback if validation or swap fails
8	Release lock
```

**Code:**

```

import os import json from pathlib import Path from contextlib import suppress

class ConfigManager: def init(self, config_dir): self.config_dir = config_dir self.lock_file_path = Path(self.config_dir) / 'config.lock' self.temp_config_dir = Path('/tmp/new_config') self.current_config_path = Path(self.config_dir) / 'current_config.json'

CODE
Copy
def reload_config(self, new_config_path):
    # Step 1: Acquire lock
    with suppress(FileExistsError):
        open(self.lock_file_path, 'w').close()

    try:
        # Step 2: Validate the new config
        self.validate_config(new_config_path)

        # Step 3: Create a temporary directory for the new configuration files
        if not os.path.exists(self.temp_config_dir):
            os.makedirs(self.temp_config_dir)

        # Step 4: Copy new config to temp dir (assuming JSON/YAML files)
        with open(self.current_config_path, 'r') as current_file:
            current_content = json.load(current_file)
        with open(self.temp_config_dir / 'current_config.json', 'w') as temp_file:
            json.dump(current_content, temp_file)

        # Step 5: Replace old config files with new ones
        for file in self.current_config_path.iterdir():
            if file.is_file():
                (self.temp_config_dir / file.name).write_text(file.read_text())

        # Step 6: Swap configuration directories
        os.rename(self.config_dir, f"{self.config_dir}_old")
        os.rename(self.temp_config_dir, self.config_dir)

    except Exception as e:
        print(f"Failed to reload config: {e}")
        # Step 7: Rollback if validation or swap fails
        rollback()

    finally:
        # Step 8: Release lock
        os.remove(self.lock_file_path)
CODE
Copy

```python
def validate_config(config_path):
    with open(config_path, 'r') as f:
        config = json.load(f)

    required_keys = ['api_key', 'timeout']
    if not all(key in config for key in required_keys):
        raise ValueError("Config is missing required keys")

    # Add more validation logic here
```

**Example input / output:**

```text
Given the following input:

- `config_dir` = `/etc/my_service/conf`
- `new_config_path` = `/tmp/new_config/new_config.json`

The expected output would be:
1. A lock is acquired in `/etc/my_service/conf/config.lock`.
2. The current configuration (`/etc/my_service/conf/current_config.json`) is validated and copied to `/tmp/new_config`.
3. New config files from `new_config_path` are replaced with the old ones.
4. The new directory is renamed to `/etc/my_service/conf`, replacing the old one.

If validation fails, the service reverts back to the original configuration without any downtime.
```

**Data:**

```yaml
# Example config files (current_config.json and new_config.json)
api_key: abc123
timeout: 5000

api_key: def456
timeout: 7000
```

**Flow:**

1. Yes
2. No
3. Acquire Lock
4. Validate Config
5. Create Temp Dir
6. Copy Old Config to Temp Dir
7. Replace Old Config with NewOnes
8. Rename Temp Dir to CurrentConfig Dir
9. Release Lock
10. Validation Failed?
11. Rollback
12. Service Reload Complete

