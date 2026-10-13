---
type: "practice"
difficulty: "Deep"
---

# A ConfigMap stores a full application config as a YAML string under a single key. What are the failure modes of embedding YAML-in-YAML and how do you keep the inner indentation intact through Helm templating?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring Helm Values](<../../../../TOOLS/YAML/YAML_Core/Refactoring_Helm_Values.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When storing an entire configuration in a ConfigMap as a single YAML string, there is a common pitfall known as "YAML-in-YAML," where one YAML document is embedded within another. This can lead to issues with indentation and overall structure, which might break the application or Helm templates if not handled correctly.

#### Triaging the Question

The question asks us to identify failure modes associated with embedding YAML content directly into a ConfigMap key and how to preserve proper indentation through Helm templating. The core of the problem is understanding how YAML parsing works and ensuring that embedded structures are rendered properly during template expansion.

#### Assumptions
We assume the application expects the entire configuration in a structured format, which is stored as a single string in a ConfigMap.
We use Helm to manage our Kubernetes resources and templates.

#### First-Principles Solution
1. YAML Structure and Indentation

YAML requires proper indentation for nested structures. If you embed one YAML document inside another without careful attention to nesting, the parser will misinterpret the structure, leading to incorrect or incomplete configurations.

For example:

YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |
    api:
      host: myapp.example.com
      port: 8080

    services:
      - name: service-a
        url: https://service-a.example.com

In this case, the services array might not be recognized correctly due to improper nesting.

2. Helm Templating and Indentation

Helm templates can help manage complex structures by allowing you to dynamically generate YAML files. However, if you embed raw YAML strings directly, you need to ensure that the template engine correctly handles the indentation and structure.

To keep the inner indentation intact, use Helm's indent function or manually manage spaces. Here’s an example using a Helm template:

YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |
{{- indent 2 $ | nindent 2 -}}
api:
    host: myapp.example.com
    port: 8080

    services:
      - name: service-a
        url: https://service-a.example.com

The indent and nindent functions help maintain the correct indentation. The indent function adds spaces to each line, while nindent ensures that subsequent lines are properly indented.

3. Example Workflow

Let’s walk through a dry-run of this approach:

Original YAML Content:

YAML
api:
    host: myapp.example.com
    port: 8080

    services:
      - name: service-a
        url: https://service-a.example.com

Using Helm Template:

YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |

{{- indent 2 $ | nindent 2 -}} api: host: myapp.example.com port: 8080

CODE
services:
  - name: service-a
    url: https://service-a.example.com
```

3. Generated Output: ```yaml apiVersion: v1 kind: ConfigMap metadata: name: app-config data: config.yaml: | api: host: myapp.example.com port: 8080

CODE
      services:
        - name: service-a
          url: https://service-a.example.com
```

This ensures that the inner indentation is maintained correctly.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To avoid common pitfalls, always ensure that YAML structures are properly nested and indented. Using Helm’s template functions like indent and nindent can help manage complex configurations stored as strings in ConfigMaps.

#### Common Mistakes
Improper Indentation: Misaligned or missing indentation can cause the configuration to be invalid.
Over-reliance on Raw Strings: Directly embedding raw YAML without template functions can lead to structural issues.
Failure to Validate Output: Not validating the generated output can result in subtle errors that only manifest during runtime.
Final Thoughts

By understanding and carefully managing indentation, you can ensure that your configurations remain robust and maintainable, even when stored as embedded YAML strings in ConfigMaps. Always test your Helm templates with different scenarios to catch potential issues early.

**Code:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |
{{- indent 2 $ | nindent 2 -}}
api:
    host: myapp.example.com
    port: 8080

    services:
      - name: service-a
        url: https://service-a.example.com
```

**Example input / output:**

```
Input:
YAML
Copy
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |

{{- indent 2 $ | nindent 2 -}} api: host: myapp.example.com port: 8080

CODE
Copy
services:
  - name: service-a
    url: https://service-a.example.com
```
Output:
YAML
Copy
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |
  api:
      host: myapp.example.com
      port: 8080

      services:
        - name: service-a
          url: https://service-a.example.com
```

**Data:**

```yaml
# Example of generated output
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |
  api:
      host: myapp.example.com
      port: 8080

      services:
        - name: service-a
          url: https://service-a.example.com
```

