---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect and prevent duplicate keys in YAML across a large manifest set, given many parsers silently keep the last value rather than error?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Indentation Pitfalls](<../../../../TOOLS/YAML/YAML_Core/Indentation_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of detecting and preventing duplicate keys in YAML manifests, we need to understand that most YAML parsers behave by overwriting values with the same key later in the file. This behavior can lead to unexpected outcomes if not properly managed. Our goal is to ensure all keys are unique while maintaining the integrity of our configuration files.

The question involves both static and dynamic detection mechanisms, which we will explore from first principles.

**Comparison:**

```
Manifest File	Services	Databases
manifest1.yaml	web (port 80)	db1 (host localhost)
manifest2.yaml	api (port 443)	db1 (host localhost)
```

**Code:**

```python
import yaml

def load_yaml(file_path):
    with open(file_path) as file:
        return yaml.safe_load(file)
```

**Example input / output:**

```

Let's consider the input manifest files manifest1.yaml and manifest2.yaml. When loaded, a naive parser might produce:

YAML
Copy
services:
- name: web
- name: api
databases:
- name: db1
  host: localhost

Here, the key 'name' in the services section is duplicated with 'web' followed by 'api', which could be undesirable.
```

**Data:**

```yaml
manifest1.yaml:
  services:
    - name: web
      port: 80
  databases:
    - name: db1
      host: localhost

manifest2.yaml:
  services:
    - name: api
      port: 443
  databases:
    - name: db1
      host: localhost
```

**Flow:**

1. Yes
2. No
3. Start
4. Read YAML File
5. Parse Success?
6. Check for Duplicate Keys
7. Mark as Valid
8. Handle Parsing Error
9. End with Error
10. Save to Database/ManifestStore
11. End with Validated Manifest

